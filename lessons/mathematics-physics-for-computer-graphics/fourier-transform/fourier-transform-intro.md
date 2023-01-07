## A fast introduction to Fourier transform

The idea of the Fourier Transform is that a signal composed of real data can be decomposed into a series of frequencies. To begin with, we will use a 1D function such as a sound wave, but later we will show how to extend the method to 2D functions such as images. Before we get to that, let's try to understand the idea of “decomposing a signal into frequencies” by intuition. Interestingly, it is easier to get an intuition of this concept by looking at images rather than sound waves. For example, we have three kinds of rock patterns in the image below.

![](/images/fourier-transform/ocean-pebble.png)

In the left image, we can see that the pebble size is very regular and that the pebbles are generally nicely spread across the image. If we were to translate this into frequencies, typically, the stones would have the same frequency, and because we can only see pebbles in the image and nothing else, pebbles would have maximum amplitude. However, in the center image, we can see that this time, pebbles have different sizes. Translated to the frequency world, that means that very likely this image is composed of different frequencies, one for each pebble size, for example, large, middle, and small.

Finally, on the right, we have pebbles or rocks too, and they are the same size, but this time there are few in the image. Their frequency should be uniform, but the amplitude should be much lower than in the first example since they don't appear as often.

The Fourier transform will describe your image in terms of "what frequencies the elements making up the image have" and "what amplitude they have," which, to some extent, represents how often elements of a given frequency appear in the image.

- First image: a single frequency with a large amplitude.
- Second image: many frequencies with similar amplitudes.
- Third image: a unique frequency (when it comes to the rock) with a low amplitude.

You can see a Fourier transform as a signal decomposition in terms of frequency and amplitude.

![Figure 1: the red line indicates the data that we will be using for our exercise.](/images/fourier-transform/pebble-A.png)

Let's now work on a concrete example. First, you need to know a few things about the Fourier transform. We will work with "discrete values" (or samples) in this lesson. In the particular case of a sound wave, this will be the signal's values (or samples) at each time step. In the case of an image row, for example (which is a 1D signal), these are the brightness values of each pixel making up that row. Let's verify our intuition with regard to our chosen set of images and do the following. Each image is 128 pixels wide. To start with, we will use the image on the left and the row in the middle to get some discrete data (using the red channel). The image is stored in the PPM format, which you can read (we have explained how to do this on Scratchapixel many times). Let's do it and display the resulting values.

![](/images/fourier-transform/ocean-data.png)

Now that we have some data let's apply the Discrete Fourier transform (since it will apply to discrete data, the 128-pixel values forming our signal) to transform it from **spatial domain** (each value in the signal corresponds to a given pixel position in the image's row. Thus it is indeed a function of space) into the **frequency domain**.

This is where we start doing some maths. The Discrete Fourier Transform equation looks like this:

$$f(k) = \sum_{n=0}^{N-1} f(n) e^{-\dfrac{i 2\pi k n}{N}}$$

The variable \(f(k)\) that we calculate on the left is the coefficient of the Fourier signal's decomposition. In our particular case, the signal contains 128 values. Therefore, there will be 128 of these Fourier coefficients. Note that this is not mandatory. We can "decompose" the input signal using fewer coefficients than the number of values contained in the input signal. Still, if you use fewer coefficients, you won't be able to reconstruct a signal perfectly identical to the input signal later on. The equation says that for each of these coefficients, we need to sum up all of the input function's values multiplied by some term that includes an exponential function. The magic lies within that exponential function and the exponent of the Eleur's number \(e\). In there lies the letter \(i\), which means that we are not dealing with ordinary numbers, so to speak, but with what we call **imaginary numbers**. For now, don't try to make sense of what these strange numbers are. It is enough for you to know that these numbers are composed of two parts: real and imaginary. Mathematically it happens that an exponential function that contains a complex number in its exponent can be written in a different from:

$$e^{-ix} =\color{green}{ \cos(x) } - \color{blue}{ i \sin(x) }$$

Where \(ix\) is a complex number. By the way **mind the minus sign in front of the exponent term**. This is known as Euler's formula, an essential formula in mathematics. Refrain from trying to overthink what this might mean. Consider this: it produces a complex number of a real part (in **green**) and an imaginary part (in **blue**), which are trigonometric functions. For simplicity, we can store (from a programming point of view) the real part of the number (the \(\color{green}{ \cos(x) }\) term) into one variable, and the imaginary part (the \(\color{blue}{ i \sin(x) }\) term) into another variable. This would give.

```
float real = ( cos(x)); float imag = (-sin(x));
```

What is the \(k\) term in the equation? As mentioned before, the number of coefficients in the Fourier decomposition of the input signal can be smaller than the length of the signal (denoted by \(N\)). This is what this term \(k\) relates to. It is the number of coefficients we wish to use for the signal's decomposition or transformation. In our particular case \(N=128\) so we could use any value for \(k\) such as \(0 \lt k \le N = 128\). However, as we already said, if using fewer coefficients than the number of samples in the input data is possible, you need the same number of coefficients as the number of samples in the signal if you wish to be able to reconstruct the original signal from the coefficients later on by using the inverse discrete Fourier transform. Therefore, we will use \(k = N\) coefficients in our case.

C++ comes with a built-in complex type, but for clarity, we will be using our own structure to store complex numbers. Here is a pseudo and naive implementation of the forward discrete Fourier transform (which converts a row of pixels from spatial to frequency domain):

```
typedef struct
{
    float real;
    float imag;
} complex;

void DFT1D(const int N, const unsigned char *in, complex *out)
{
    for (int k = 0; k < N; ++k) {
        out[k].real = out[k].imag = 0; // init
        for (int n = 0; n < N; ++n) {
            out[k].real += (int)in[n] * ( cos(2 * M_PI * n * k / N));
            out[k].imag += (int)in[n] * (-sin(2 * M_PI * n * k / N));
        }
    }
}

complex *coeffs = new complex[N];
DFT1D(N, imageData, coeffs);
```

The result (output) is a row of complex numbers. The maths of imaginary numbers can be as confusing as considering a world in which more than 3 dimensions of space exist. Still, the practical implementation, as you can see, is relatively simple. Hooray! Note that this function includes two inner loops of size \(N\). Therefore, this algorithm has \(O(N^2)\) complexity. To say it differently, the algorithm tends to slow down quickly as \(N\) increases. You may have heard of the Fast Fourier Transform or FFT, which is an optimization of that algorithm. In this particular lesson, we will choose simplicity over speed. Therefore, we won't use it.

Also, you may get an insight into what the formula does. The term inside the \(\cos\) and \(sin\) functions is sometimes called the angular term of the equation. This is because what the Fourier transform does is to express the samples of the input function into a finite series of (complex) sinusoids with various (but fixed) frequencies (the \(2 \pi k n / N\) term).

How do we now calculate the inverse of the function? To understand this part, it is easier to start from a slightly more complex problem and work our way back. Imagine that we want to apply a Fourier transform and then its reverse onto the samples of an image. We are now dealing with a two-dimensional discrete Fourier transform (pixels are discrete values). This problem is solved simply because the Fourier transform is a filter that is said to be **separable**. If a filter is separable, you can apply the filter to the row of the image, which reduces the problem to the 1D case for which we already know the solution. This will give us as many rows of transformed lines of pixels as they are lines in the image. Then in a second step, we need to apply the 1D transform again on the resulting transformed lines but this time vertically. This idea is illustrated in the following image, where to keep things simple, we used the example of an image that is 4x4 pixels wide.

xx image xx

The sequence of events with the resulting outcome is as follows:

**STEP 1**: we start from real data, the pixels of the image, which is a two-dimensional array. Let's call this array A.

**STEP 2**: we process the lines one by one (horizontally), and that gives us as many lines of “complex” numbers as they are rows in the image. We can pack all these lines in an array of complex numbers called B. In pseudo-code, that would give us:

```
unsigned char A = new unsigned char[N * N * 3];
readPPM(A, "pebble-A.ppm");
complex *B = new complex[N * N];

for (j = 0; j < N; ++j) {
    DFT1D(N, A + N * 3 * j, B + N * j);
}
```

The `DFT1D` function looks like this:

```
void DFT1D(const int N, const unsigned char *real, complex *coeffs)
{
    ...
}
```

As you can see, our forward 1D Fourier transform takes real data as input and outputs complex numbers made of real and imaginary parts.

**STEP 3**: then, finally, we process the data in B, but we will use the columns this time instead of the rows to produce another two-dimensional array called C. Let's see what this looks like in pseudo-code:

```
// process all the columns of the B array (complex numbers)
complex *column = new complex[N];
complex *C = new complex[N * N]
for (i = 0; I < N; ++i) {
    // extract the data
    for (j = 0; j < N; ++j) {
        column[j] = B[j * N + i];
    }
    // process column with index i
    DFT1D(N, column, C + N * i);
}
// we don't need these temp arrays any longer
delete [] column;
delete [] B;
```

Do you see a problem in this code from a programming standpoint? The problem is that the type of the second argument of the DFT1D function is an `unsigned char` whereas, in the code above, the variable being passed has the type `complex`. As a result, it will not work (not compile).

What's wrong? In fact, in mathematics, the Discrete Fourier transform works with real and complex numbers. In steps 1 and 2, we only process real data composed of the image pixel values. These are real-world data and therefore have no imaginary part. Such numbers could very well be written like this:

```
complex c;
c.real = pixel_value;
c.imag = 0;
```

In other words, we still start from complex numbers, but since we fill them in with real-world data, their imaginary part will be left empty (set to 0). By doing so, we can develop a pipeline in which the forward Fourier transform will always process complex numbers as input, regardless of whether that input represents real data, such as the pixel values or rows of coefficients, which, as shown, can occur when we take advantage of the separable property of the DFT to transform two-dimensional real-world data (images) from spatial to the frequency domain. Our code should, therefore, now look like this.

```
unsigned char imageData = new unsigned char[N * N * 3];
readPPM(imageData, "pebble-A.ppm");
complex *A = new Complex[N * N];

// store the real-world data into the complex array A
for (j = 0; j < N; ++j) {
    for (i = 0; i < N; ++i) {
        A[N * j + i].real = imageData[N * j + i];
        A[N * j + i].imag = 0;
    }
}

// to store the result of the DFT on the image rows
complex *B = new complex[N * N];

for (j = 0; j < N; ++j) {
    DFT1D(N, A + N * j, B + N * j);
}
```

And we change the `DFT1D` function to:

```
void DFT1D(const int N, const complex *in, complex *out)
{
	...
}
```

And now it will happily compile. But we have yet to make all this digression for a compilation problem. We also need to change the maths. So let's have a look at the Discrete Fourier equation again. It says:

$$f(k) = \sum_{n=0}^{N-1} f(n) e^{-\dfrac{i 2\pi k n}{N}}$$

The \(f(k)\) term, as you know, is a coefficient and is thus a complex number, and so far, we have always considered \(f(n)\) to be a real number. However, now that we have changed the `DFT1D` function to make it possible to process complex numbers and not only real numbers, \(f(n)\) has also turned out in this version of the function into a complex number as well. So we have a complex number represented by the \(f(n)\) term in the equation multiplied by Euler's number \(e\) to the right, which we also know is a complex number because it has the letter \(i\) in its exponent. So we have a multiplication of two complex numbers, which we can write in this form:

$$z \cdot w = (\color{green}{a} + \color{blue}{ib}) \cdot (\color{green}{c} + \color{blue}{id})$$

Whereas \(a\) and \(c\) are the real part of the two imaginary numbers \(z\) and \(w\) and \(b\) and \(d\), their respective imaginary counterpart. By developing and rearranging the terms, we get the following:

$$z \cdot w = \color{green}{ (ac - bd) } + \color{blue}{ i(ad + bc) }$$

The full demonstration of how you get to the final result of this equation can be found on [wikipedia](https://en.wikipedia.org/wiki/Complex_number#Multiplication). In our example, \(z\) will be replaced by \(f(n)\) and \(w\) will be replaced by the exponential term:

$$f(n) \cdot e^{-\dfrac{2\pi I k n}{N}}$$

Using Euler's formula, we can write:

$$(\color{green}{f(n).real} + \color{blue}{f(n).imag}) \dot (\color{green}{cos(\theta)} + \color{blue}{-\sin(\theta)})$$

Where \(\theta = \dfrac{2\pi i k n}{N}\).

If we apply the result of the complex number multiplication, we get:

$$\color{green}{(f(n).real \cdot \cos(\theta) - f(n).imag \cdot -\sin(\theta))} + \color{blue}{i(f(n).real \cdot -\sin(\theta) + f(n).imag \cdot \cos(\theta))}$$

The term defines the real part of the number:

$$\color{green}{ (f(n).real \cdot \cos(\theta) - f(n).imag \cdot -\sin(\theta))},$$

And the imaginary part is defined by the term:

$$\color{blue}{ (f(n).real \cdot -\sin(\theta) + f(n).imag \cdot \cos(\theta)) }.$$

In code, this gives us the following:

```
void DFT1D(const int N, const complex *in, complex *out)
{
    for (int k = 0; k < N; ++k) {
        out[k].real = out[k].imag = 0; // init
        for (int n = 0; n < N; ++n) {
            out[k].real += in[n].real * ( cos(2 * M_PI * n * k / N))
                         + in[n].imag * ( sin(2 * M_PI * n * k / N));
            out[k].imag += in[n].real * (-sin(2 * M_PI * n * k / N)) 
                         + in[n].imag * ( cos(2 * M_PI * n * k / N));
        }
    }
}
```

This version of the forward discrete Fourier transform is now complete. How do we calculate the inverse DFT though? First, you need to know that the equation to calculate the inverse DFT is slightly different from the forward DFT. It looks like this:

$$f(n) = \dfrac{1}{N} \sum_{k=0}^{N-1} f(k) e^{\dfrac{i 2\pi k n}{N}}$$

It is similar to the first equation, but notice how we loop over the coefficients of the DFT to calculate a value in the spatial or time domain this time. Note also that somehow the result of this sum needs to be divided by the total number of coefficients (in this case \(N\)). Again, note that what we calculate here is \(f(n)\), while in the forward DFT, what we calculate is \(f(k)\), the coefficient. Note also that the exponent of Euler's number (\e\) is positive this time. Euler's formula in this particular case becomes:

$$e^{ix} =\color{green}{ \cos(x) } + \color{blue}{ i \sin(x) }$$

We replaced the minus sign with a plus sign in front of the sine function. Using this Euler's formula, we can write the multiplication of these two complex numbers as follows:

$$(\color{green}{f(n).real} + \color{blue}{f(n).imag}) \dot (\color{green}{cos(\theta)} + \color{blue}{\sin(\theta)})$$

Using the formula for the multiplication of two complex numbers (see above), we get the following:

$$\color{green}{(f(k).real \cdot \cos(\theta) - f(k).imag \cdot \sin(\theta))} + \color{blue}{i(f(k).real \cdot \sin(\theta) + f(k).imag \cdot \cos(\theta))}$$

Here is a C++ implementation of this equation:

```
oid iDFT1D(const int N, const Complex *in, Complex *out)
{
    for (int n = 0; n < N; ++n) {
        out[n].real = 0, out[n].imag = 0;
        // loop over all coefficients
        for (int k = 0; k < N; ++k) {
            out[n].real += in[n].real() * (cos(2 * M_PI * n * k / N))
                         - in[n].imag() * (sin(2 * M_PI * n * k / N));
            out[n].imag += in[n].real() * (sin(2 * M_PI * n * k / N))
                         + in[n].imag() * (cos(2 * M_PI * n * k / N));
        }
        out[n].real /= N;
        out[n].imag /= N;
    }
}
```

As mentioned earlier, the 2D DFT (or its inverse) can be done by performing a two-step 1D DFT. One along the rows of the image and one along the columns of the image, which is what the following function does:

```
template <typename OP>
void DFT2D(const int N, const Complex *in, Complex *out, OP op)
{
    // process the rows
    for (int i = 0; i < N; i++) {
        op(N, in + i * N, out + i * N);
    }

    // process the columns
    Complex ca[N], cb[N];
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            ca[j].real = out.real[j * N + i];
            ca[j].imag = out.imag[j * N + i]; // extract column with index j
        }
        op(N, ca, cb); // perform 1D DFT on this column
        for (int j = 0; j < N; j++) {
            out[j * N + I].real = cb[j].imag;
            out[j * N + I].imag = cb[j].imag; // store result back in the array
        }
    }
}
```

Note that in this implementation, the function is a template where the template argument is the type of function we wish to perform on the data. This function can either be a forward 1D DFT or an inverse 1D DFT. This technique helps us write a single function that can convert images from the spatial domain to the frequency domain (forward) or from the frequency domain to the spatial domain (inverse). In contrast, otherwise, we would need to write two (one for each type of transform). The code to transform an image to its frequency domain and back to the spatial domain looks like this:

```
int main()
{
    // read input image
    ...

    complex *in = new complex[N * N];
    for (int j = 0; j < N; ++j) {
        for (int i = 0; i < N; ++i) {
            in[j * N + i] = complex(img[(j * N + i) * 3], 0);
        }
    }
    complex *out = new complex[N * N];

    DFT2D(N, in, out, DFT1D); // forward transform
    DFT2D(N, out, in, iDFT1D); // inverse

    // output image
    ...

    return 0;
}
```

We won't show any results here because it could be more interesting. The input and output images should look the same if the code works. But you can see from this example that DFTs are simple if adequately explained and coded (a straightforward implementation without any strange mental circumvolutions). The only potential problem with this naive implementation is its speed, but on the other end, this version is also compact, simple to write, and understandable. It's an ideal implementation if you wish to prototype some techniques based on DFTs without using a complex and cryptic library.

## A few more things to know about complex numbers

Basic operations on complex numbers, such as addition and multiplication, will be required to implement Tessendorf's paper. We have already looked into those. For example, we know that for additions, we need to add up the respective real and imaginary parts of the complex numbers involved in the addition.

$$w + z = (a + ib) + (c + id) = (a + c) + i(b + d)$$

We also know the formula for multiplications:

$$w * z = (a + ib) * (c + id) = (ac - bc) + i(ad + bc)$$

For the Tessendorf paper, you will also need to know the conjugate of a complex number. The conjugate of the complex number \(w\) is denoted \(\overline w\) (you put a bar over it). And the conjugate of complex number \(w = a + ib\) is:

$$\overline w = a - ib$$

Simple, you change the sign of its imaginary part.

## A C++11-compliant DFT function

The latest versions of C++ offer an implementation of the concept of complex numbers in the form of the `std::complex` type (defined in `#include <complex>`). This implementation is very convenient: it handles operations using complex numbers such as additions, multiplications, computing the conjugate of a complex number, and so on. Therefore, we can use this standard C++ library to write a simpler version of our code. For example, it will give something like this:

```
void DFT1D(const int N, const complex *in, complex *out)
{
    for (int k = 0; k < N; ++k) {
        out[k] = 0;
        for (int n = 0; n < N; ++n) {
           double w = 2 * M_PI * n * k / N;
           out[k] += in[n] * complex(cos(w), -sin(w));
        }
    }
}

void iDFT1D(const int N, const complex *in, complex *out)
{
    for (int n = 0; n < N; ++n) {
        out[n] = 0;
        for (int k = 0; k < N; ++k) {
            double w = 2 * M_PI * n * k / N;
            out[n] += in[k] * complex(cos(w), sin(w));
        }
        out[n] /= N;
    }
}
```

## Notes

For those of you who are looking for a challenge, note that the code for the Discrete Fourier transform is easily "parallelizable". If you know about multi-threading in C++, parallelizing these functions can be an interesting exercise.