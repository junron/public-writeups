# SCAndal

>  Your aim is to identify the 6 removed values, sort them in ascending order, then your flag would be `flag{num[0],num[1],num[2],num[3],num[4],num[5]}` 
>
> For example:
> if the missing numbers are [1, 69, 42, 169, 142, 242], the flag would be `flag{1,42,69,142,169,242}`.

We are given a C file that contains a custom binary search implementation, as well as some sort of serial communications. 

```c
int main(void) {
  platform_init();
  init_uart();
  trigger_setup();
  simpleserial_init();

  int key = 0xdeadbeef; // ofc it is a dummy value

  srand(key);
  remove_six();

  simpleserial_addcmd('s', 1, search);

  while(1) simpleserial_get();
  return 0;
}
```

After initializing the serial communications, the function calls `remove_six`. 

```c
void remove_six(void) {
  for (uint8_t i = 1 ; i <= 6; i++) {
    uint8_t rep = rand();
    arr[256] = arr[rep];
    memmove(&arr[rep], &arr[rep+1], (256-rep)*sizeof(arr[0]));
  }
}
```

This function randomly picks and removes 6 elements from the array. The array is a 257 element array containing the numbers 0 to 255 in order, with a 0 at the end. Our task is to find out which 6 elements were removed. The program then configures the `search` function to handle the command `s`. 

```c
uint8_t search(uint8_t* data, uint8_t dlen)
{
  trigger_high();
  binary_search(0, 249, data[0]);
  trigger_low();
  return 0;
}
```

This function calls `trigger_high`, binary searches a value in the array, then calls `trigger_low`. This might be manipulating the serial communications. If we can get the time between `trigger_high` and `trigger_low`, we could find out how long it takes to binary search each number, thus leaking information about the elements in the array.

This data is provided in the `traces.npy` file, generated by the `get_traces.py` file. It contains a 2D array of dimensions `(256,3000)`, with float values in `(-0.5, 0.5)`. I plotted the first row in matplotlib to visualize it:

![image-20211226063532868](./images/plot1.png)

We can clearly see a region in the middle where the magnitude of the lines are consistently much greater than at the ends. This should be the time period where the binary search is running. I decided to extract this time period by finding the first and last value where the reading is below -0.4, and finding the number of readings between them. Again, I plotted this with matplotlib.

![image-20211226064030435](./images/plot2.png)

I was incredibly surprised by this result as I didn't expect it to work so well. We can clearly see 6 values where the time taken is less than 2100. I think I was just really lucky to have chosen -0.4 as the threshold value as no other number has such a clear result. 

![image-20211226064543051](./images/results.png)



What was weird to me was the timings for the removed values were much lower than expected. In my tests using python, the removed values required more calls to `binary_search`. I've also attached the Jupyter notebook [here](files/solve.ipynb) as well as the [trace data](./files/traces.npy) so you can play with it.



