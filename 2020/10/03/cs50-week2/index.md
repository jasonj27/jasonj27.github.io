# CS50 2020 Week2: Arrays Problem Answers

<!--more-->
## [Readability](https://cs50.harvard.edu/x/2020/psets/2/readability/)
```c
#include <cs50.h>
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <math.h>

int main(void)
{
    string input = get_string("Text: ");

    float letters = 0, words = 0, sentences = 0, index;

    int input_length = strlen(input);

    for (int i = 0; i < input_length; i++)
    {
        if (isalpha(input[i]) != 0)
        {
            letters++;
        }
        else if (isspace(input[i]))
        {
            words++;
        }
        else if ((input[i]) == 46 || (input[i]) == 21 || (input[i]) == 63)
        {
            sentences++;
        }
    }
    words++;

    index = roundf(0.0588 * (letters / words) * 100 - 0.296 * (sentences / words) * 100 - 15.8);


    if (index <= 1)
    {
        printf("Before Grade 1\n");
    }
    else if (index >= 2 && index <= 15)
    {
        printf("Grade %i\n", (int)index);
    }
    else if (index >= 16)
    {
        printf("Grade 16+\n");
    }

}
```

## [Substitution](https://cs50.harvard.edu/x/2020/psets/2/substitution/)
```c
#include <stdio.h>
#include <cs50.h>
#include <string.h>
#include <ctype.h>

int main(int argc, string argv[])
{
    if (argc != 2)
    {
        printf("Usage: ./substitution KEY\n");
        return 1;
    }

    char key[26][2] = {{'a', '0'}, {'b', '0'}, {'c', '0'}, {'d', '0'}, {'e', '0'}, {'f', '0'}, {'g', '0'}, {'h', '0'}, {'i', '0'}, {'j', '0'}, {'k', '0'}, {'l', '0'}, {'m', '0'}, {'n', '0'}, {'o', '0'}, {'p', '0'}, {'q', '0'}, {'r', '0'}, {'s', '0'}, {'t', '0'}, {'u', '0'}, {'v', '0'}, {'w', '0'}, {'x', '0'}, {'y', '0'}, {'z', '0'}

    };

    if (strlen(argv[1]) != 26)
    {
        printf("KEY must contain 26 characters.\n");
        return 1;
    }

    for (int i = 0; i < 26; i++)
    {
        if (isalpha(argv[1][i]) == 0)
        {
            printf("Key must only contain alphabetic characters.\n");
            return 1;
        }

        for (int j = 0; j < 26; j++)
        {
            if (key[j][1] == tolower(argv[1][i]))
            {
                printf("Key must not contain repeated characters.\n");
                return 1;
            }
        }

        key[i][1] = tolower(argv[1][i]);
    }

    string s = get_string("plaintext: ");

    for (int i = 0; i < strlen(s); i++)
    {
        if (isupper(s[i]))
        {
            for (int j = 0; j < 26; j++)
            {
                if (toupper(key[j][0]) == s[i])
                {
                    s[i] = toupper(key[j][1]);
                    break;
                }
            }
        }

        if (islower(s[i]))
        {
            for (int j = 0; j < 26; j++)
            {
                if ((key[j][0]) == s[i])
                {
                    s[i] = (key[j][1]);
                    break;
                }
            }
        }
    }

    printf("ciphertext: %s\n", s);

    return 0;
}
```

