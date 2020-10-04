# CS50 2020 Week4: Memery Problem Answers

<!--more-->
## [Filter](https://cs50.harvard.edu/x/2020/psets/4/filter/more)
```c
#include "helpers.h"
#include <math.h>
#include <string.h>

// Convert image to grayscale
void grayscale(int height, int width, RGBTRIPLE image[height][width])
{
    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            float pixel = (float)(image[i][j].rgbtRed + image[i][j].rgbtGreen + image[i][j].rgbtBlue) / 3;
            pixel = roundf(pixel);

            image[i][j].rgbtRed = pixel;
            image[i][j].rgbtGreen = pixel;
            image[i][j].rgbtBlue = pixel;
        }
    }
    return;
}

// Reflect image horizontally
void reflect(int height, int width, RGBTRIPLE image[height][width])
{
    for (int i = 0; i < height; i++)
    {
        RGBTRIPLE aux[width];
        memcpy(aux, image[i], sizeof(RGBTRIPLE) * width);
        for (int j = 0; j < width; j++)
        {
            image[i][j].rgbtRed = aux[width - j - 1].rgbtRed;
            image[i][j].rgbtGreen = aux[width - j - 1].rgbtGreen;
            image[i][j].rgbtBlue = aux[width - j - 1].rgbtBlue;
        }
    }
    return;
}

// Blur image
void blur(int height, int width, RGBTRIPLE image[height][width])
{
    RGBTRIPLE aux[height][width];
    memcpy(aux, image, sizeof(RGBTRIPLE) * height * width);

    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            int r = 0, g = 0, b = 0;
            float pixel_count = 0;

            for (int k = -1; k < 2; k++)
            {
                for (int l = -1; l < 2; l++)
                {
                    if (!(i + k < 0 || i + k >= height || j + l < 0 || j + l >= width))
                    {
                        r += aux[i + k][j + l].rgbtRed;
                        g += aux[i + k][j + l].rgbtGreen;
                        b += aux[i + k][j + l].rgbtBlue;
                        pixel_count++;
                    }
                }
            }

            float pixel_blur_r = roundf((float)r / pixel_count);
            float pixel_blur_g = roundf((float)g / pixel_count);
            float pixel_blur_b = roundf((float)b / pixel_count);

            image[i][j].rgbtRed = pixel_blur_r;
            image[i][j].rgbtGreen = pixel_blur_g;
            image[i][j].rgbtBlue = pixel_blur_b;
        }
    }
    return;
}

// Detect edges
void edges(int height, int width, RGBTRIPLE image[height][width])
{
    RGBTRIPLE aux[height][width];
    memcpy(aux, image, sizeof(RGBTRIPLE) * height * width);

    int g_x[3][3] =
    {
        {-1, 0, 1},
        {-2, 0, 2},
        {-1, 0, 1}
    };

    int g_y[3][3] =
    {
        {-1, -2, -1},
        {0, 0, 0},
        {1, 2, 1}
    };

    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            int rx = 0, gx = 0, bx = 0;
            int ry = 0, gy = 0, by = 0;

            for (int k = -1; k < 2; k++)
            {
                for (int l = -1; l < 2; l++)
                {
                    if (!(i + k < 0 || i + k >= height || j + l < 0 || j + l >= width))
                    {
                        rx += aux[i + k][j + l].rgbtRed * g_x[k + 1][l + 1];
                        gx += aux[i + k][j + l].rgbtGreen * g_x[k + 1][l + 1];
                        bx += aux[i + k][j + l].rgbtBlue * g_x[k + 1][l + 1];

                        ry += aux[i + k][j + l].rgbtRed * g_y[k + 1][l + 1];
                        gy += aux[i + k][j + l].rgbtGreen * g_y[k + 1][l + 1];
                        by += aux[i + k][j + l].rgbtBlue * g_y[k + 1][l + 1];
                    }
                }
            }

            float r = roundf(sqrt((float)(rx * rx + ry * ry)));
            float g = roundf(sqrt((float)(gx * gx + gy * gy)));
            float b = roundf(sqrt((float)(bx * bx + by * by)));

            r = r > 255 ? 255 : r;
            g = g > 255 ? 255 : g;
            b = b > 255 ? 255 : b;

            image[i][j].rgbtRed = r;
            image[i][j].rgbtGreen = g;
            image[i][j].rgbtBlue = b;
        }
    }
    return;
}
```
## [Recovery](https://cs50.harvard.edu/x/2020/psets/4/recover)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

typedef uint8_t BYTE;

int main(int argc, char *argv[])
{
    BYTE buffer[512];
    char filename[8];
    int n = 0;
    FILE *img;

    if (argc != 2)
    {
        printf("Usage: ./recover image\n");
        return 1;
    }

    FILE *f = fopen(argv[1], "r");

    if (!f)
    {
        printf("Error open file\n");
        return 1;
    }

    // loop for read file
    while (1)
    {
        // Read 512 bytes into a buffer
        int bytesread = fread(buffer, sizeof(BYTE), 512, f);

        // if start a new JPEG
        if (buffer[0] == 0xff && buffer[1] == 0xd8 && buffer[2] == 0xff && (buffer[3] & 0xf0) == 0xe0)
        {
            if (n == 0) // If first JPEG
            {
                sprintf(filename, "%03i.jpg", n);
                n++;
                img = fopen(filename, "w");
                fwrite(buffer, sizeof(BYTE), bytesread, img);
            }
            else // not first JPEG
            {
                fclose(img);
                sprintf(filename, "%03i.jpg", n);
                n++;
                img = fopen(filename, "w");
                fwrite(buffer, sizeof(BYTE), bytesread, img);
            }
        }
        else
        {
            if (n != 0) // If already found JPEG
            {
                fwrite(buffer, sizeof(BYTE), bytesread, img);
            }
        }

        // which means file end, fread read only size of bytesread
        if (bytesread != 512)
        {
            // write last read to file
            fwrite(buffer, sizeof(BYTE), bytesread, img);
            fclose(img);
            break;
        }
    }

    fclose(f);
    return 0;
}
```

