# CS50 2020 Week1: C Problem Answers

<!--more-->
## [mario](https://cs50.harvard.edu/x/2020/psets/1/mario/more/)
```c
#include <stdio.h>
#include <cs50.h>

int main(void)
{
    int height = -1;
    while (height > 8 || height < 1)
    {
        height = get_int("Height: ? ");
    }

    for (int i = 0, j = height - 1; i < height; i++, j--)
    //i for height, j for space before each level
    {
        for (int k = 0; k < 2; k++)
        //loop for right half and left half
        {
            if (k == 0)
            //for left half
            {
                if (j != 0)
                //if Height is one, skip output space before each level
                {
                    for (int l = 0; l < j; l++)
                    //print space before each level
                    {
                        printf(" ");
                    }
                }
                for (int m = 0; m <= i; m++)
                //print left half #
                {
                    printf("#");
                }
                printf("  ");
                //space between left half and right half
            }
            else
            {
                for (int m = 0; m <= i; m++)
                //print right half #
                {
                    printf("#");
                }
            }
        }
        printf("\n"); //new line before next level
    }
}
```
## [credit](https://cs50.harvard.edu/x/2020/psets/1/credit/)
```c
#include <stdio.h>
#include <cs50.h>

int main(void)
{
    long number = get_long("Number: "), first_two;

    int digit = 0, chk1 = 0, chk2 = 0;
    //chk2 is second-to-last digit

    for (long i = number; i != 0;)
    {
        digit++;
        i /= 10;
    }

    for (long i = number; i >= 100; i /= 10)
    {
        first_two = i / 10;
    }


    for (int i = 0; number != 0; i++)
    {
        if (i % 2 == 0)
        {
            chk1 += number % 10;
            number /= 10;
        }
        else
        {
            int temp = number % 10 * 2;
            chk2 += temp / 10;
            chk2 += temp % 10;
            number /= 10;

        }
    }



    if ((chk1 + chk2) % 10 != 0)
    {
        printf("INVALID\n");
    }
    else
    {
        if (first_two == 34 || first_two  == 37)
        {
            if (digit == 15)
            {
                printf("AMEX\n");
            }
            else
            {
                printf("INVALID\n");
            }
        }
        else if (first_two >= 51 && first_two <= 55)
        {
            if (digit == 16)
            {
                printf("MASTERCARD\n");
            }
            else
            {
                printf("INVALID\n");
            }
        }
        else if (first_two / 10 == 4)
        {
            if (digit == 16 || digit == 13)
            {
                printf("VISA\n");
            }
            else
            {
                printf("INVALID\n");
            }
        }
        else
        {
            printf("INVALID\n");
        }
    }
}
```
