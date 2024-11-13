# Effective-Encryption-Tool-
A highly effective encryption tool with more features in later versions. 

```#include <stdio.h>
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
