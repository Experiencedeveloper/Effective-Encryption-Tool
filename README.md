# Effective Encryption Tool

## Overview
The Effective Encryption Tool is a versatile program designed for encrypting and decrypting messages using multiple cipher methods. This tool provides a user-friendly interface and ensures secure communication through various encryption techniques.

## Features
### v1.0: "Initial Release, Alpha or Beta Development Stage, Great Encryption"
- **Substitution Cipher**: Encrypts and decrypts messages by substituting each letter with a different one based on a predefined key.
- **Random Key Generation**: Generates a random substitution key for encryption.
- **Custom Key Input**: Allows users to input their own substitution key.
- **Interactive User Interface**: Uses `ncurses` for a text-based UI, providing a smooth user experience.

### v2.0: "Best Encryption, Beta Development Stage"
- **Substitution Cipher**
- **Caesar Cipher**: Encrypts and decrypts messages by shifting each letter by a specified number of places in the alphabet.
- **Vigenère Cipher**: Encrypts and decrypts messages using a keyword.
- **Enhanced ncurses-based UI**: Improved user interface for better usability.
- **Improved Key Validation**: Ensures custom keys are valid and secure.

### v3.0: "Possibly Complete, Top-notch Encryption"
- **All features from v2.0**
- **AES-256 Encryption**: Provides robust encryption using the Advanced Encryption Standard.

## Installation
1. **Requirements**:
    - A C compiler (e.g., GCC)
    - `ncurses` library installed on your system
    - OpenSSL library for AES-256 (for v3.0)

2. **Installation Steps**:
    ```sh
    sudo apt-get install gcc ncurses-dev libssl-dev
    ```

3. **Compile the Program**:
    ```sh
    gcc -o encryption_tool encryption_tool.c -lncurses -lssl -lcrypto
    ```

## Usage
1. **Running the Program**:
    ```sh
    ./encryption_tool
    ```

2. **Key Generation**:
    - Press 'r' to generate a random key.
    - Press 'c' to enter a custom key.

3. **Message Encryption**:
    - Enter your message when prompted.
    - The tool will display the encrypted message.

4. **Message Decryption**:
    - The tool automatically decrypts the encrypted message and displays it.

## Cipher Methods
### Substitution Cipher
- Substitutes each letter in the plaintext with a different letter based on a predefined key.
- Example Key: "QWERTYUIOPASDFGHJKLZXCVBNM"
- Example: "HELLO" -> "ITSSG" (using the above key)

### Caesar Cipher
- (Coming in v2.0) Shifts each letter in the plaintext by a specified number of places.
- Example Shift: 3
- Example: "HELLO" -> "KHOOR"

### Vigenère Cipher
- (Coming in v2.0) Uses a keyword to shift letters, providing more security than the Caesar cipher.
- Example Keyword: "KEY"
- Example: "HELLO" -> "RIJVS" (using the keyword "KEY")

### AES-256 Encryption
- (Coming in v3.0) Uses the Advanced Encryption Standard for robust encryption.
- Example: Encrypting "HELLO" will produce a secure, non-readable ciphertext.

## Code Structure
### encryption_tool.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
#include <ctype.h>
#include <ncurses.h>

// Function to generate a random substitution key
void generate_random_key(char* key) {
    int alphabet[26] = {0};
    for (int i = 0; i < 26; i++) {
        int randIndex;
        do {
            randIndex = rand() % 26;
        } while (alphabet[randIndex] != 0);
        alphabet[randIndex] = 1;
        key[i] = 'A' + randIndex;
    }
    key[26] = '\0';
}

// Function to encrypt the message
void encrypt(const char* message, const char* key, char* encrypted) {
    for (size_t i = 0; i < strlen(message); i++) {
        if (isalpha(message[i])) {
            char base = islower(message[i]) ? 'a' : 'A';
            encrypted[i] = key[message[i] - base];
        } else {
            encrypted[i] = message[i];
        }
    }
    encrypted[strlen(message)] = '\0';
}

// Function to decrypt the message
void decrypt(const char* encrypted, const char* key, char* decrypted) {
    for (size_t i = 0; i < strlen(encrypted); i++) {
        if (isalpha(encrypted[i])) {
            char base = islower(encrypted[i]) ? 'a' : 'A';
            for (int j = 0; j < 26; j++) {
                if (key[j] == encrypted[i]) {
                    decrypted[i] = base + j;
                    break;
                }
            }
        } else {
            decrypted[i] = encrypted[i];
        }
    }
    decrypted[strlen(encrypted)] = '\0';
}

int main() {
    char* message = (char*)malloc(256 * sizeof(char));
    char key[27];
    char* encrypted = (char*)malloc(256 * sizeof(char));
    char* decrypted = (char*)malloc(256 * sizeof(char));

    if (message == NULL || encrypted == NULL || decrypted == NULL) {
        printf("Memory allocation failed\n");
        return 1;
    }

    // Initialize random seed
    srand(time(NULL));

    // Initialize ncurses
    initscr();
    cbreak();
    echo();  // Enable echo to make characters visible while typing
    keypad(stdscr, TRUE);
    
    // Create windows for input and output
    WINDOW *inputwin = newwin(10, 50, 0, 0);
    WINDOW *outputwin = newwin(20, 50, 12, 0);  // Increase the height of the output window
    box(inputwin, 0, 0);
    box(outputwin, 0, 0);
    mvwprintw(inputwin, 0, 1, " Input ");
    mvwprintw(outputwin, 0, 1, " Output ");
    refresh();
    wrefresh(inputwin);
    wrefresh(outputwin);

    // Loop for key generation input
    while (1) {
        mvwprintw(inputwin, 1, 1, "Press 'r' for random key, 'c' for custom key: ");
        wrefresh(inputwin);
        int ch = wgetch(inputwin);

        if (ch == 'r') {
            generate_random_key(key);
            mvwprintw(inputwin, 2, 1, "Generated Key: %s", key);
            break;
        } else if (ch == 'c') {
            mvwprintw(inputwin, 2, 1, "Enter the custom key (26 unique letters): ");
            wrefresh(inputwin);
            wgetnstr(inputwin, key, 26);
            if (strlen(key) != 26) {
                mvwprintw(inputwin, 3, 1, "Invalid key length. Press any key to exit...");
                wrefresh(inputwin);
                wgetch(inputwin);
                endwin();
                return 1;
            }
            break;
        } else {
            mvwprintw(inputwin, 2, 1, "Invalid option. Press any key to try again...");
            wrefresh(inputwin);
            wgetch(inputwin);
        }
        wrefresh(inputwin);
    }

    // Loop for message input
    while (1) {
        // Prompt for the message
        mvwprintw(inputwin, 4, 1, "Enter the message (type 'q' to quit): ");
        wrefresh(inputwin);
        wgetnstr(inputwin, message, 255);

        // Check if the user wants to exit
        if (strcmp(message, "q") == 0) {
            break;
        }

        // Encrypt and display the message
        encrypt(message, key, encrypted);
        mvwprintw(outputwin, 1, 1, "Encrypted message: %s", encrypted);
        wrefresh(outputwin);

        // Display a gap of 5 lines before the decrypted message
        for (int i = 2; i < 7; i++) {
            mvwprintw(outputwin, i, 1, "");
        }

        // Decrypt and display the message
        decrypt(encrypted, key, decrypted);
        mvwprintw(outputwin, 7, 1, "Decrypted message: %s", decrypted);
        wrefresh(outputwin);
    }

    // End ncurses
    endwin();

    // Free allocated memory
    free(message);
    free(encrypted);
    free(decrypted);

    return 0;
}
```

## Future Enhancements
- Adding more encryption methods (e.g., Vigenère cipher, Caesar cipher in v2.0, AES-256 in v3.0).
- Improving the user interface with more features and better navigation.
- Enhancing error handling and input validation.
- Providing unit tests and example scripts for better usability and reliability.

## Contribution
- Feel free to fork the repository and submit pull requests for any improvements or new features.

## License
- This project is licensed under the MIT License.
