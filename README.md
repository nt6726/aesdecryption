# nathantsaiaes
Name: Nathan Tsai

Note on running the program: To get it to run with 
a custom program name simply compile it with: gcc aes.c -o <program name>. Then it can be run with whatever program name: 
./<program name> --inputfile $inputfile_name ...etc.

The main function first takes in the options provided by the user on the command line when running the program. 
It saves these options as char* so they can be used when reading and writing to specific file names. Next, 
the program checks the keysize that the user specified. If the keysize is 128, the AES encryption/decryption will run for 10 rounds. 
If the keysize is 256, the AES encryption/decryption will run for 14 rounds. Next, the program reads in the bytes of the keyfile and the 
input file by the file names specified by the user. The number of bytes of the input (the state) is then checked. If the number of bytes 
is not a multiple of 16, 0s are padded to the end of the current state, and the new padded state is used in the encryption. The total number of 
bytes of the state is then checked to see if it is greater than 16 bytes. If it is greater than 16 bytes, then encryption and decryption are performed on 
the state 16 bytes at a time, with the first 16 encrypted/decrypted bytes being combined with the next 16 encrypted/decrypted bytes and so on. 
Otherwise, encryption/decryption is performed on the 16 bytes of the state as normal, depending on if the state has been padded with 0s or not. 
The program then outputs the encrypted/decrypted bytes to the output file name specified by the user.

Key Expansion:
When encrypting/decrypting the state with the round key, the program checks the current state, the keysize and the number of rounds (depending on 
if we are using AES-128 or AES-256). When encrypting/decrypting the key is first expanded using the key expansion and scheduling algorithm. 
The number of bytes it gets expanded to is the number of bytes of the original key plus the number of rounds multiplied by 16:
original key size + (16 * number of rounds)
The expandKey() function does the key expansion. It is passed in the original key, a new unsigned char array that will hold the 
expanded key, with a size that is determined by using the previous step of getting the key expansion size. The first 16 or 32 bytes are 
the original key, depending on the original key size. To calculate the rest of the expanded key's bytes, an algorithm is used on each four 
bytes of the original key.
In this algorithm, firstly, the four bytes are shifted to the left by one byte, so the data in byte 0 becomes the data in byte 1. 
The data in byte 1 becomes the data in byte 2, byte 2 becomes byte 3, and byte 3 becomes byte 0. Then, a function called 
subWord() is performed. This function compares the four bytes with a given array (that is also used for subBytes() in encryption) and it uses 
the contents of each of the four bytes as indeces for this array. The values at these indeces in the array are then assigned as the new four 
bytes. It is to be noted that in AES-256 key expansion, all of the steps of the key expansion algorithm are done every 32 bytes, but the 
subWord() step is done every 16 bytes, so it is done twice every 32 bytes. The next step in the algorithm is to take the first byte of the 
four and raise it to the power of an index of a different array, called rcon (for round constant), used in key scheduling, subtracted by 1. 
This is done just to make each key schedule a little different from the previous one. To do this, 
simply XOR the first byte of the four by the value of the rcon array at the index specified, which starts at 1, and is incremented by 1 
every time 16 bytes. Finally, the four bytes are XOR'd to the new array storing the expanded key. The final expanded key is store in a 
variable called e_key.

Encryption Flow:
The encryption algorithm first takes the original key and performs the function addRoundKey() to it. 
addRoundKey() simply takes the state and XOR's each element with each element in the round key that is passed to the function as an argument. 
In this case, the first round key is just the original key that the user had input as the bytes of the key file. Then, for the encryption 
rounds, the rounds start at 1 and end at the number of rounds-1 because the last round is unique from the others. For each round, first subBytes() 
is performed. This function uses each byte in the state to use as an index for a given array, named sbox in this 
program. The value of the sbox array at this index is then assigned to the current state, replacing the original byte that was 
used as an index. It does this for each of the 16 bytes that are currently being encrypted. Next, the shiftRows() function is performed on the 
state that is being encrypted. Thinking of the current state as a four by four matrix,the shiftRows() function shifts the bytes in the 
first row (row 0) to the left zero bytes (or zero columns), hence the first row does not need to be changed. It shifts the second row (row 1) 
down the matrix to the left by one column (1 byte), the third row to the left by 2 bytes, and fourth row to the left by 3 bytes. Indexing is 
important here because, for example, element in row 0 column 1 would actually have an index of 4 in the current state's unsigned char*. Thus, 
it may be easier to go through each column of the 4x4 state matrix, shifting each row down by its corresponding number of bytes. 
Next, after shifting all of the rows, mixColumns() is performed on the current state. mixColumns(), thinking again of the current state as a 
4x4 matrix, replaces each column with its value multiplied by a special 4x4 matrix, the Galois field. This different matrix is given, but multiplying 
the bytes together requires some more given matrices that specify what each byte should turn out to be when multipled by 2 or 3. Thus, if a byte 
needs to be multiplied, it is passed into the multiplication matrices as an index, and its new value is the value of the matrix at that index. 
Every four bytes of the state are multiplied (through XORing) by the special matrix, and the result of mixColumns() is then assigned back to the 
current state. After mixColumns(), addRoundKey is then applied to the current state again, but this time using the next 16 bytes in the expanded 
key for each round. The next 16 bytes of the expanded key are determined by taking the (current round number * 16) + expanded key. After the 
rounds have looped through to (number of rounds - 1), the last round is performed. In the last round, mixColumns is not performed, so it is 
just subBytes, shiftRows, and addRoundKey that are performed on the current state. In the last, round addRoundKey is performed with the last 
16 bytes of the expanded key. The program then outputs the encrypted bytes to the output file specified by the user.

Decryption Flow:
The decryption algorithm is just the encryption algorithm but inversed. This means there are specialized functions that perform the inverses 
of the functions used in encryption. Firstly, key expansion is again performed on the original key that the user input. Then, since addRoundKey() 
is simply performing XOR operations, the function is the inverse of itself. addRoundKey() is performed using the current state that the user 
is trying to decrypt and the last 16 bytes of the extended key. Then, the rounds start at the number of rounds - 1 because like with encryption, 
the last round is unique. In each looping round, invShiftRows() is first performed on the current state. This function does the same thing as shiftRows() 
except instead of shifting the rows (thinking of the current state as a 4x4 matrix) to the left, this time they are being shifted to the right. 
So row 0 is shifted to the right by 0 bytes (0 columns), row 1 is shifted to the right by 1 byte, row 2 to the right by 2 bytes, and row 3 to the 
right by 3 bytes. Again, indexing is important here because the current state being decrypted is not actually a 4x4 matrix, and it is instead an 
unsigned char*. The result of invShiftRows() is then reassigned back to the current state. Next in the decryption flow is invSubBytes(). This time, 
instead of using each byte of the current state as an index for the sbox array, a different array, the sbox_inv array is used. The value of each 
byte in the current state is used as the index for the sbox_inv array, and each byte in the current state is replaced with the resulting byte in the 
sbox_inv array. Then, addRoundKey() is performed again with the current state and the next 16 bytes in the expanded key. Then, invMixColumns() is 
performed on the current state. invMixColumns() uses a different Galois field matrix to multiply to the current state (by using XOR). This time, 
the bytes are multipled by either 0x09, 0x0b, 0x0d, or 0x0e. Again, there are particular arrays used in these multiplications. Each four bytes 
of the current state are used as indeces for these particular multiplication arrays, and the resulting value is assigned as the new byte value for 
the current state. After all of the columns have been inversely mixed, the result is reassigned back to the current state. After the intial rounds 
have completed, the final round of decryption is also special. It does not include invMixColumns(). It performs invShiftRows() on the current state, 
then invSubBytes() on the current state, then addRoundKey() on the first 16 bytes of the expanded key, which is also the original key before expansion. 
After all of these steps have completed, the decryption is done, and the resulting decrypted bytes are output to the output file specified by the user.
