
# Use of "printf()" with STM32 

In this blog, i have demonstrated how to use printf() function to print it on the serial window of STM32CubeIDE itself.

In this i have used STM32F407 discovery board, STM32CubeIDE.




## Steps 

- Create new project in STM32CubeIDE

- Go to Core -> Src -> syscalls.c

```c
/* Variables */
extern int __io_putchar(int ch) __attribute__((weak));
extern int __io_getchar(void) __attribute__((weak));

/* Added Manually */
// Debug Exception & Monitor Control Register Base Address
#define DEMCR               *((volatile uint32_t*) 0xE000EDFCU)

// ITM Register Address
#define ITM_STIMULUS_PORT0  *((volatile uint32_t*) 0xE0000000)
#define ITM_TRACE_EN        *((volatile uint32_t*) 0xE0000E00)

void ITM_SendChar (uint8_t ch)
{
	// Enable TRCENA
	DEMCR  |= (1<<24);

	// Enable Stimulus Port 0
	ITM_TRACE_EN |= (1<<0);

	// Read FIFO Status in bit[0]
	while (!(ITM_STIMULUS_PORT0 &1));

	//Write to ITM Stimulus Port0
	ITM_STIMULUS_PORT0 = ch;
}
/* Added Manually End */

```
- Copy the code commented inside `Added Manually` part and paste it below the `Variables` comment as shown above.

- In the same file i.e syscalls.c 

``` c
__attribute__((weak)) int _write(int file, char *ptr, int len)
{
  (void)file;
  int DataIdx;

  for (DataIdx = 0; DataIdx < len; DataIdx++)
  {
//    __io_putchar(*ptr++);
    ITM_SendChar(*ptr++);
  }
  return len;
}
```

- Modify or Paste this code as shown above.

- Go to Core -> Src -> main.c 

```c

/* USER CODE BEGIN Includes */
#include <stdlib.h>
#include <stdio.h>
/* USER CODE END Includes */

```

- Add these two header files in the `USER CODE BEGIN Includes section`, as these is added Manually.




## Demo of using printf()

- In the demo part i will show how to use printf function in the code.

- I will show the counter incrementation and display it on SWV IT Data Console

```c
/* USER CODE BEGIN PV */
uint8_t count = 0;
/* USER CODE END PV */

```
- Create a private variable `count` and initalize it to `0`.

- In the while loop write a logic to increment the counter and print it in the SWD IT Data Console

```c
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	printf("The Counter Value is : %d \r\n", count);
	count++;
	fflush(stdout);
	HAL_Delay(1000);
  }
  /* USER CODE END 3 */
```
- Save and build the Project.

- Run as -> STM32 Application.

- Click Debug Icon, to enter STM32 in debug mode

- Go to Window -> Show View -> SWV -> SWV ITM Data Console `(Note: This steps to be done in Debug mode only)`

- Below SWV ITM Data Console tab will open, click on `Configure Trace` , select Port 0 in `ITM Stimulus Ports` check box and then click OK.

- Next to it, click on `Start Trace` button.

- Click Resume button or the shortcut for it is F8 on keyboard.

- Now the printf statement is visible in SWV ITM Data Console.