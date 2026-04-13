This is the first actual CTF challenge in reverse engineering that I was able to solve! Very excited to share this writeup.

Basic checks:-

<img width="2267" height="82" alt="image" src="https://github.com/user-attachments/assets/c8170e3c-717d-4fad-8ad4-488ed3f3fbee" />

The file is a 64 bit ELF, which we will try to reverse. The file is 'stripped' which means the name of functions and variables will all be obfuscated

<img width="954" height="55" alt="image" src="https://github.com/user-attachments/assets/46f14e7d-c15d-40bc-872f-672fa8a89fbc" />

The input prompt is "Please make a compatible password:" and the code exits with the wrong password.

Lets disassemble the executable in `Ghidra` for static analysis:-

There are around 9-10 functions in this executable once we open the file in `Ghidra`

To find the entrypoint or the `main` function, we navigate to the `entry` function

```
void processEntry entry(undefined8 param_1,undefined8 par am_2)

{
  undefined1 auStack_8 [8];
  
  __libc_start_main(input_func,param_2,&stack0x00000008,0, 0,param_1,auStack_8);
  do {
                    /* WARNING: Do nothing block with infinite loop  */
  } while( true );
}

```

Above is the pseudo code for `entry` function. In the `__libc_start_main`, the first parameter is the name of our `main` function. I have renamed it to `input_func`

###### Analysis of `input_func()`:-

Pseudo code

```
undefined8 input_func(void)

{
  size_t sVar1;
  char local_28 [27];
  undefined1 local_d;
  
  printf("Please make a compatible password: ");
  __isoc99_scanf("%27[^\n]",local_28);
  local_d = 0;
  sVar1 = strlen(local_28);
  if (sVar1 == 0x1b) {
    calc_one_func(local_28);
  }
  return 0;
}
 
```

```
int input_func(void)

{
  size_t input_length; //length of the user_input
  char input_buffer [27];  //buffer where we store the user input
  int local_d; //not sure why this is used, it is set to '0' in line 10
  
  printf("Please make a compatible password: ");
  __isoc99_scanf("%27[^\n]",input_buffer);  // this is basically scanf but with input validation, the width is set to 27 bytes and we cannot include "\n" which is new line when we press enter key
  local_d = 0;
  input_length = strlen(input_buffer);
  if (input_length == 0x1b) {  // 0x1b is 27 in integers
    calc_one_func(input_buffer);
  }
  return 0;
}
```

From this function, we got to know that our input must be exactly 27 bytes long without including the '\n' which appears when we hit enter
If our string length is 27 i.e. 0x1b, then `calc_one` function will be called giving our` input_buffer` as the argument otherwise our code exits

###### Analysis of `calc_one()`:-

pseudo-code

```
void calc_one_func(char *param_1)

{
  calc_two_func(param_1,5);
  calc_two_func(param_1,6);
  if (param_1[0xb] == 'o') {
    calc_two_func(param_1,0xd);
    if (param_1[0xe] == 'R') {
      calc_two_func(param_1,3);
      calc_two_func(param_1,0x18);
      if ((*param_1 == -0x65) && ((byte)(param_1[0x1a] + 0x 8dU) < 5)) {
        calc_two_func(param_1,10);
        if ((param_1[8] == 'Y') && ((param_1[0xb] == 'Y' && (( byte)(param_1[0xc] + 0x8cU) < 4)))) {
          calc_two_func(param_1,7);
          if ((param_1[0x14] == -0x4b) && (param_1[0xd] == 's ')) {
            flag_func(param_1);
          }
        }
      }
    }
  }
  return;
}
```

```
void calc_one_func(char *input_buff) //takes the argument as pointer to the input we typed in "input_func"

{
  calc_two_func(input_buff,5);  //calling calc_two with odd second param
  calc_two_func(input_buff,6);  //calling calc_two with even second param
  if (input_buff[0xb] == 'o') {
    calc_two_func(input_buff,0xd);
    if (input_buff[0xe] == 'R') {
      calc_two_func(input_buff,3);
      calc_two_func(input_buff,0x18);
      if ((*input_buff == -0x65) && ((byte)(input_buff[0x1a] + 0x8dU) < 5)) {
        calc_two_func(input_buff,10);
        if ((input_buff[8] == 'Y') && ((input_buff[0xb] == 'Y' && (( byte)(input_buff[0xc] + 0x8cU) < 4)))) {
          calc_two_func(input_buff,7);
          if ((input_buff[0x14] == -0x4b) && (input_buff[0xd] == 's')) {
            flag_func(input_buff);
          }
        }
      }
    }
  }
  return;
}
```

we need to satisfy all the 'if' conditions to get to the `flag_func()` present in line 17, we are calling` calc_two()` multiple times during this
which will in turn call `calc_three()` and the calculation is too huge to try and reverse manually

In line 12, if `((*input_buff == -0x65) && ((byte)(input_buff[0x1a] + 0x8dU) < 5))`

First condition:-
`(*input_buff == -0x65)`
`-0x65` is negative 101 or '-101' in integer and it will be written in 2s complement method which is `0x9B`
we are checking if the first byte present in the buffer is 0x9B

Second condition:-
`((byte)(input_buff[0x1a] + 0x8dU) < 5)`
`0x1a` is 26 in integers which means the last character in our string, we add `0x8d` to it which is 141,
After adding we check if the resulting value is less than 5 i.e. one of {0,1,2,3,4}
And since we are typecasting to byte which means value must be between 0 to 255, if the value exceeds that, we simply do `% 256`
modulo 256, and this should be between 0 to 4
If you check the ASCII table, it basically means check if the values are `s,t,u,v,w`

Multiple similar conditions are present in this function, one thing we are sure is that we need to satisfy all of them to proceed to the `flag_func()`

###### Analysis of `calc_two()`:-

pseudo-code

```
void calc_two_func(long param_1,uint param_2)

{
  int iVar1;
  undefined4 local_10;
  undefined4 local_c;

  if ((param_2 & 1) == 0) {
    for (local_c = 0; local_c < (int)param_2; local_c = local_c + 1) {
      iVar1 = (int)(local_c * param_2) % 0x1b;
      *(char *)(param_1 + iVar1) = *(char *)(param_1 + iVar1 ) + (char)param_2;
    }
    calc_three_func(param_1,param_2);
  }
  else {
    calc_three_func(param_1,param_2);
    for (local_10 = 0; local_10 < (int)param_2; local_10 = loca l_10 + 1) {
      iVar1 = (int)(local_10 + param_2) % 0x1b;
      *(char *)(param_1 + iVar1) = *(char *)(param_1 + iVar1 ) - (char)param_2;
    }
  }
  return;
}
```

```
void calc_two_func(long input_string,uint param_2)

{
  int iVar1;
  int j; //counter for second for loop
  int i; //counter for first for loop

  // 'if' condition will be satisfied when param_2 is even
  // 'else' condition when param_2 is odd
  
  if ((param_2 & 1) == 0) {  //param_2 is even
    for (i = 0; i < (int)param_2; i = i + 1) {
      iVar1 = (int)(i * param_2) % 0x1b;
      *(char *)(input_string + iVar1) = *(char *)(input_string + iVar1 ) + (char)param_2;
    }
    calc_three_func(input_string,param_2);
  }
  else { //param_2 is odd
    calc_three_func(input_string,param_2);
    for (j = 0; j < (int)param_2; j = j + 1) {
      iVar1 = (int)(j + param_2) % 0x1b; //big gotcha in this line, (j + param_2), in the 'if' block it was 'i * param_2'
      *(char *)(input_string + iVar1) = *(char *)(input_string + iVar1 ) - (char)param_2;
    }
  }
  return;
}
```

"if" block:-

Example, lets take` param_2 as '6'`
iVar1 will take values =`{0,6,12,18,24,3}`

`*(char *)(input_string + iVar1)`

In this part, `input_string` has stored the address of the first character of our user typed input string
`input_string+ iVar1` means move ahead by that many spaces in memory, example if `iVar1 = 6`, value present at `input_string[6]` is increased by `ascii 6`
`(char *) `is used to say that the type is a pointer to a single character
`*` in the outside means it is a dereference and modifies the actual value at that memory which is pointing to a character

So, for` iVar 0,6,12,18,24,3` the values at these indices of our string are increase by char(6)
After all iterations, `calc_three()` function is called

Similarly in the 'else' block:-
Look carefully, here iVar1 is calculated in a different manner, it is `(j + param_2)`
So, for `param_2 = 5`
`iVar1 = {5,6,7,8,9}`
First, `calc_three()` function is called and then, similar calculations are carried out as 'if' block.
However, major difference here is that instead of adding, we are subtracting value from the specific places in `input_string`

###### Analysis of `calc_three()`:-

pseudo-code

```
void calc_three_func(char *param_1,int param_2)

{
  char local_38 [40];
  undefined4 local_10;
  int local_c;

  local_10 = 0;
  strcpy(local_38,param_1);
  for (local_c = 0; local_c < 0x1b; local_c = local_c + 1) {
    param_1[(local_c + param_2) % 0x1b] = local_38[local_c];
  }
  param_1[0x1b] = '\0';
  return;
}
```

```
void calc_three_func(char *input_str,int param_2)

{
  char copy_of_str [40]; //copy of our string
  int local_10; //not sure why this is used, set to '0' in line 9
  int k; //counter used in for loop
  
  local_10 = 0;
  strcpy(copy_of_str,input_str);  //copies the string in second argument to the first argument
  for (k = 0; k < 0x1b; k = k + 1) {
    input_str[(k + param_2) % 0x1b] = copy_of_str[k];
  }
  input_str[0x1b] = '\0'; //setting the byte after 27 input bytes as null bytes input_str[27] is the 28th character
  return;
}
```

In this 'for' loop, our original string is modified using the copied string as reference
value of 'k' iterates from '0' to '26' and increments by '1' every step
original string index of `(k + second param) % 0x1b` is replaced with copy string index `k`
On calculation we can find out that this function is actually right-shifting our original string by the value `param_2`
Example, if `param_2 = 5` the original string will get modified in such a way that instead of starting at` index 0`, it will start at` index 5`, means `original[5] = copy[0]`
All the overflowing characters after index 26 in original string will be appended from`index 0`instead

example:-   ` abcdefgh`, shift right 2 becomes `ghabcde`

Analysis of `flag_func()`:-

pseudo code

```
void flag_func(char *param_1)

{
  int iVar1;
  long lVar2;
  char local_38;
  undefined1 local_37;
  undefined1 local_36;
  char local_35;
  undefined1 local_34;
  undefined1 local_33;
  char local_32;
  undefined1 local_31;
  undefined1 local_30;
  char local_2f;
  undefined1 local_2e;
  undefined1 local_2d;
  undefined1 local_2c;
  char local_2b;
  char local_2a;
  char local_29;
  char local_28;
  char local_27;
  char local_26;
  char local_25;
  char local_24;
  char local_23;
  char local_22;
  undefined1 local_21;
  int local_20;
  int local_1c;
  int local_18;
  int local_14;
  FILE *local_10;
  
  local_2b = *param_1 + -0x21;
  local_22 = (param_1[0x1a] + '\x06') * -2;
  local_2a = param_1[1] + -0x20;
  local_23 = param_1[8] + -7;
  local_29 = param_1[2] + -0x28;
  local_24 = param_1[9] + '\x14';
  local_28 = (param_1[3] + '\x04') * '\x02';
  local_25 = param_1[10] + '\b';
  local_27 = param_1[0xc] + '\x1c';
  local_26 = param_1[0xb] + -0x66;
  local_21 = 0;
  iVar1 = strcmp(&local_2b,s_README.txt_00404060);
  if (iVar1 != 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  local_10 = fopen(&local_2b,"rb");
  if (local_10 == (FILE *)0x0) {
    perror("fopen");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  fread(&local_2c,1,1,local_10);
  local_2f = calc_four_func((int)(char)(-2 - param_1[5] / '\x02' ));
  local_2e = calc_four_func((int)(char)(param_1[6] + '\x04'));
  local_2d = 0;
  lVar2 = strtol(&local_2f,(char **)0x0,0x10);
  local_14 = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_32 = calc_four_func((int)(char)(param_1[7] + -0x2b));
  local_31 = calc_four_func((int)(char)(param_1[0x19] + -0x3 1));
  local_30 = 0;
  lVar2 = strtol(&local_32,(char **)0x0,0x10);
  local_18 = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_35 = calc_four_func((int)(char)(param_1[0x18] + '\x0 5'));
  local_34 = calc_four_func((int)(char)('\b' - param_1[0x17] / '\x02'));
  local_33 = 0;
  lVar2 = strtol(&local_35,(char **)0x0,0x10);
  local_1c = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_38 = calc_four_func((int)(char)(param_1[0x16] + -0xf ));
  local_37 = calc_four_func((int)(char)(param_1[0x15] + -0x3 d));
  local_36 = 0;
  lVar2 = strtol(&local_38,(char **)0x0,0x10);
  local_20 = (int)lVar2;
  if ((((local_1c == 0x61) && (local_18 == 0x34)) && (local_ 14 == 0x57)) && (local_20 == 0x29)) {
    printf("You have entered the flag");
  }
  return;
}
```

This is a complex looking function but we will go step by step:

```
  char local_2b;
  char local_2a;
  char local_29;
  char local_28;
  char local_27;
  char local_26;
  char local_25;
  char local_24;
  char local_23;
  char local_22;
  undefined1 local_21;
  int local_20;
  int local_1c;
  int local_18;
  int local_14;
  FILE *local_10;
  
  local_2b = *input_string + -0x21;
  local_22 = (input_string[0x1a] + '\x06') * -2;
  local_2a = input_string[1] + -0x20;
  local_23 = input_string[8] + -7;
  local_29 = input_string[2] + -0x28;
  local_24 = input_string[9] + '\x14';
  local_28 = (input_string[3] + '\x04') * '\x02';
  local_25 = input_string[10] + '\b';
  local_27 = input_string[0xc] + '\x1c';
  local_26 = input_string[0xb] + -0x66;
  local_21 = 0;
  iVar1 = strcmp(&local_2b,s_README.txt_00404060);    //Here we are checking if both the strings are equal
```

 `strcmp()` will compare both the strings till it hits null byte, if you check ghidra the value stored in `'s_README.txt_00404060'` is `"README.txt\x00"`
  which is 11 bytes in total counting the null byte

<img width="472" height="101" alt="image" src="https://github.com/user-attachments/assets/554cce81-dcf2-447f-90b0-8286a46dacf0" />

  `&local_2b` is the address where` local_2b` starts, but it will not stop at local_2b since its a char, it will continue to the following contiguous memories till it hits null
  
  **Note-** We are not changing the actual values present in the input string, we are only performing calculations using variables local to this function
  
  Check the stack for actual variables that will be checked
  The variables are `local_2b` through `local_21`
  You can verify this by looking at the stack for this function, `local_2b` is present at offset `'rbp - 0x2b'` hence the name
  the other functions are ex `rbp - 0x2a`, `rbp - 0x29` and so on upto `rbp - 0x21`
  
<img width="391" height="199" alt="image" src="https://github.com/user-attachments/assets/ed30bf5e-6eb5-4ac8-bf2f-f8965dd507d0" />


Always remember that the stack actually grows downwards, the top of the stack is at the lowest memory address. In the above snap, `local_2b` is present at the lowest memory which is at the top of stack and it is a `char` type. If you keep adding 1 to the stack, we will go from `local_2b` to `local_2a` since all of them are only a single byte long.

  If you count the functions, 2b,2a,29,28,27,26,25,24,23,22,21 are exactly 'eleven' which can hold `"README.txt\x00"` exactly
  We start comparison at address of` local_2b` and compare it with 'R'
  Then we move +1 in memory which means from `'rbp - 0x2b'` to `'rbp - 0x2a'`, since we are moving from lower memory addresses to higher memory
 `local_2b` is present at lowest memory address among these functions
  We can also see that in line 47, we are assigning value '0' to the last function in series `local_21`which will serve as the `null byte`

```
  if (iVar1 != 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  // If the strings do not match the program will exit with an error
```

```
  local_10 = fopen(&local_2b,"rb");  // fopen() is used to open the file which has a name starting at first argument, with 'rb' as the mode
  /*
  If there is an error in opening the file, NULL will be returned to local_10, otherwise a FILE* which points to the file in question
  &local_2b currently stores the string "README.txt\x00" as we saw in the previous part of the code
  Which means we are opening a file named "README.txt" which also present in the current working directory as the executable binary
  We need to make sure that a "README.txt" file is present in the current working directory
  'r' means we are opening file in read-mode, 'b' is there for ISO C compatibility and has no other effect
  */
  if (local_10 == (FILE *)0x0) {  //If file does not open, we enter this section of code and exit with an error.
    perror("fopen");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
```

Which basically means we need a `README.txt` file in the same directory as the binary executable and it should contain some random data which can be read. Otherwise, our code will exit.

```
  fread(&local_2c,1,1,local_10);
  /*
  size_t fread(void *buffer, size_t size, size_t count, FILE *stream);
  Above is the syntax of fread() function in C
  First arg is the place where we will be storing, second arg is the size of bytes of each block we will be reading, third arg is number
  of blocks we will be reading, fourth is the FILE from which we will be reading

  In our case this means, read one byte of data, one time, from the file pointed to by local_10 which is README.txt and store it in local_2c
  */
```

```
  local_2f = calc_four_func((int)(char)(-2 - input_string[5] / '\x02' ));
  // We are performing basic arithmetic on whatever character is present at index 5 of input_string, first we divide by 2 and then we subtract 2
  // Lets say at input_string[5], the character '8' was present which in decimal ASCII is '56'. now we do (-2 -(56/2)) = (-2 - 28) = -30
  // But in 2s complement it will be written as 0xE2, then it gets converted  to int 0xFFFFFFE2 because of the sign bit set
  // 0xE2 is written as '11100010' in binary, you can see the most significant bit which is sign bit is set
  // when we pass this to calc_four function, it takes character as an argument, so from 0xFFFFFFE2, it will take only 0xE2 which is still -30 for it
  local_2e = calc_four_func((int)(char)(input_string[6] + '\x04'));
  //Similar calculations performed here as well
  local_2d = 0;
  //local_2d is set to null byte
  lVar2 = strtol(&local_2f,(char **)0x0,0x10);
  /*
  strtol() converts string to long, it will take the string starting at address of local_2f until null byte is encountered
  We have a char stores at local_2f, we have char at local_2e and then we have NULL at local_2d
  When we check for char, we keep incrementing by 1, local_2f is present at rbp - 0x2f offset, incrementing one we get rbp - 0x2e which is local_2e and so on
  Second argument mentions NULL, means count string till NULL is encountered, which we will encounter at local_2d
  Last argument mentions 0x10 which is 16 in decimal, means convert this string to base 16 or hexadecimal
  Example calculation, lets say local_2f has '2' and local_2e has 'c' and local_2d has NULL
  To convert '2C' to hexadecimal, we do the following calculation
  The Formula: (Current Total×16)+New Digit
  first digit:- (0 x 16) + 2
  new current total = 2
  second digit:- (2 x 16) + 12
  In hex, A means 10, B means 11, C means 12 and so on till F means 16 so we have 0-9, A-F covering 16 values
  new total = 44
  If you check in python, hex(44) is 0x2C
  */
  local_14 = (int)lVar2;
  // we are converting the value to int and storing in local_14
```

```
//We repeat the same things below as well
  fread(&local_2c,1,1,local_10);
  local_32 = calc_four_func((int)(char)(input_string[7] + -0x2b));
  local_31 = calc_four_func((int)(char)(input_string[0x19] + -0x31));
  local_30 = 0;
  lVar2 = strtol(&local_32,(char **)0x0,0x10);
  local_18 = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_35 = calc_four_func((int)(char)(input_string[0x18] + '\x05'));
  local_34 = calc_four_func((int)(char)('\b' - input_string[0x17] / '\x02'));
  local_33 = 0;
  lVar2 = strtol(&local_35,(char **)0x0,0x10);
  local_1c = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_38 = calc_four_func((int)(char)(input_string[0x16] + -0xf ));
  local_37 = calc_four_func((int)(char)(input_string[0x15] + -0x3d));
  local_36 = 0;
  lVar2 = strtol(&local_38,(char **)0x0,0x10);
  local_20 = (int)lVar2;

  //Now we perform the following comparisons and if everything is correct, we will get the following output that the flag is correct
  if ((((local_1c == 0x61) && (local_18 == 0x34)) && (local_14 == 0x57)) && (local_20 == 0x29)) {
    printf("You have entered the flag");
  }
  return;
}
```

We can figure out a lot of the characters of our final transformed string which reached `flag_func()` and gave us the flag applying these constraints


###### Analysis of `calc_four()`:-

```
char calc_four_func(char param_1)

{
  if ((((param_1 < '0') || ('9' < param_1)) && ((param_1 < 'a' || ('f' < param_1)))) &&
     ((param_1 < 'A' || ('F' < param_1)))) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  return param_1;
}
```

This function is very simple, it basically states that whatever character is given to this function as a parameter
should be between '0 to 9, or 'a to f' or 'A to F'
Which means that the input character must be a valid hexadecimal character only
Any character other than that would immediately exit


Now we understand what the code exactly does and we have quite a few constraints in the `flag_func()` functions from which we can identify most of the characters in our string


###### The big picture:- 

We have a lot of functions in the given binary named 'switcheroo'
As the name suggests, we are switching multiple times between functions and its gonna be a pain in the ass to figure out the logic

Out of all the functions present, I have identified the main ones and named them according to their usage

The input prompt, where we are asked "Please make a compatible password:" is present in the "`input_func()`"

From there we move to the "`calc_one`" function
"`calc_one`" function performs a lot of calculations and calls "`calc_two`" function on many instances and the end of the final "if"
statement, it calls the "`flag_func`"

"`calc_two`" function has an "if-else" statement and it calls "`calc_three`" in both of the blocks

"`calc_three`" function does not call any function and performs some calculations

"`flag_func`" is the biggest function and it calls "`calc_four`" multiple times, and in the end there is a statement "You have entered the flag"

"`calc_four`" function does not call any other function and performs basic operations

Map thus far:-

entry: `input_func` ----> `calc_one` ----> multiple `calc_two` calls ----> `calc_three` on each `calc_two` call ----> `calc_two` calls `flag_func` -----> multiple `calc_four` calls ----> flag in the end

The working of all the functions is written as comments in the respective file of the functions


###### Calculations for flag characters:-

Trying to figure out the final string which will give us the flag:-
-----------------

index_0 - s (flag_func)
index_1 - e (flag_func)
index_2 - i (flag_func)
index_3 - 0x1e (flag_func)
index_4 - 
index_5 - ’ (right quote) or ‘ (left quote) (flag_func) - These characters cannot be used we will have to use `0x91 or 0x92` in place because of encoding format mismatch
index_6 - digit '3' (flag_func)
index_7 - ^ (flag_func)
index_8 - 0x7f (flag_func)
index_9 - ` (flag_func)
index_10 - & (flag_func)
index_11 - « (flag_func)
index_12 - 1 (flag_func)
index_13 - s (calc_one_func)
index_14
index_15
index_16
index_17
index_18
index_19
index_20 - µ (calc_one_func)
index_21 - v (flag_func)
index_22 - A (flag_func)
index_23 - ® or 'SHY' soft hyphen (flag_func)
index_24 - 1 (flag_func)
index_25 - e (flag_func)
index_26 - 0xC0 (flag_func)

-----------------
Hints from `flag_func` file

```
  local_2b = *input_string + -0x21;
  local_22 = (input_string[0x1a] + '\x06') * -2;
  local_2a = input_string[1] + -0x20;
  local_23 = input_string[8] + -7;
  local_29 = input_string[2] + -0x28;
  local_24 = input_string[9] + '\x14';
  local_28 = (input_string[3] + '\x04') * '\x02';
  local_25 = input_string[10] + '\b';
  local_27 = input_string[0xc] + '\x1c';
  local_26 = input_string[0xb] + -0x66;
  local_21 = 0;
  iVar1 = strcmp(&local_2b,s_README.txt_00404060);
```
From this we know that local_2b is the starting character of the string which ends at local_21 making exactly 11 characters including NULL at local_21
The entire string from local_2b to local_21 should be "README.txt\x00" ending in NULL shown by \x00

```
local_2b should equal 'R'
which means input_string[0] - 0x21 should equal 'R'
'R' in hex is 0x52
which means input_string[0] must be 0x52 + 0x21 which is 115 in decimal which is character 's'
```

```
local_2a = input_string[1] + -0x20;
local_2a is 'E'
input_string[1] equals 0x45 + 0x20 which is 101 in decimal which means character 'e'
```

```
local_29 = input_string[2] + -0x28;
local_29 is 'A' which is 0x41
input_string[2] is 0x41 + 0x28 which is 'i'
```

```
local_28 = (input_string[3] + '\x04') * '\x02';
local_28 is 'D' which is 0x44
doing calculations, input_string[3] comes out to be 0x1e which is not a printable character
```

```
local_27 = input_string[0xc] + '\x1c';
local_27 should be 'M' which is 0x4D
input_string[0xc] must equal decimal 49 which is character '1'
```

```
local_26 = input_string[0xb] + -0x66;
local_26 should be 'E' which is 0x45
input_string[0xb] should be 171 in decimal which is "«" in ASCII character
```

```
local_25 = input_string[10] + '\b';
local_25 must be '.' which is 0x2E
input_string[10] must be decimal 38 which is '&' in ASCII 
```

```
local_24 = input_string[9] + '\x14';
local_24 should be 't' which is 0x74
input_string[9] will be decimal 96 which is '`'
```

```
local_23 = input_string[8] + -7;
local_23 must be 'x' which is 0x78
input_string[8] will be 127 in decimal which is 0x7f in hex and is not printable character
```

```
local_22 = (input_string[0x1a] + '\x06') * -2;
local_22 should be 't' which is 0x74
The value of input_string[0x1a] must be 0xC0 (decimal 192, or -64 signed).
```

-----------------------------

hints from strtol() in flag_func()

```
local_2f = calc_four_func((int)(char)(-2 - input_string[5] / '\x02' ));
local_2e = calc_four_func((int)(char)(input_string[6] + '\x04'));
local_2d = 0;
lVar2 = strtol(&local_2f,(char **)0x0,0x10);
local_14 = (int)lVar2;
(local_14 == 0x57)
```

You can read about the working of these lines of code in flag_func file itself, we will start with the calculations directly

we know that local_14 must be 0x57
Value for this will be provided by local_2f and local_2e
Assuming local_2f to be character '5' and local_2e to be '7'
When we perform strtol() and convert them to hex, we use the following formula

sum = (current sum * 16) + new digit

for first digit which is '5':-
sum = (0 * 16) + 5 = 5

for second digit which is '7':-
sum = (5 * 16) + 7 = 87

87 in hex is written as 0x57
Which means our assumptions are correct

```
means (-2 - input_string[5] / '\x02' ) = digit '5'
53 (ascii for digit 5) = -2 -input_string[5] / 2
55 = -input_string[5] / 2
input_string[5] = -110 or 146(considering signed)
= ’ (right quote)
There is another possibility that input_string[5] might be -111 as well or 145 because 111/2 is also 55 in integer division
= ‘ (left quote)

Note:- However, right and left single quotes dont follow the modern encoding patterns so we will have to use the actual numbers directly like chr(0x91) or chr(0x92) in their place
```

<img width="217" height="112" alt="image" src="https://github.com/user-attachments/assets/a530fb48-448f-4c15-9192-e1ff374faa99" />


```
(input_string[6] + '\x04') must be digit '7'
55 = input_string[6] + 4
input_string[6] = 51 = digit '3'
```


```
local_32 = calc_four_func((int)(char)(input_string[7] + -0x2b));
local_31 = calc_four_func((int)(char)(input_string[0x19] + -0x31));
local_30 = 0;
lVar2 = strtol(&local_32,(char **)0x0,0x10);
local_18 = (int)lVar2;
(local_18 == 0x34)

Using the above logic, local_32 is digit '3' and local_31 is digit '4'
(3 * 16) + 4 = 52 = 0x34

(input_string[7] + -0x2b) = digit '3'
51 = input_string[7] - 43
input_string[7] = 94 = ^ in ASCII

(input_string[0x19] + -0x31) = digit '4'
52 = input_string[0x19] - 49
input_string[0x19] = 101 = 'e' in ASCII
```


```
local_35 = calc_four_func((int)(char)(input_string[0x18] + '\x05'));
  local_34 = calc_four_func((int)(char)('\b' - input_string[0x17] / '\x02'));
  local_33 = 0;
  lVar2 = strtol(&local_35,(char **)0x0,0x10);
  local_1c = (int)lVar2;
(local_1c == 0x61)

from above logic, local_35 must be digit '6' and local_34 must be digit '1'
(6 * 16) + 1 = 97 = 0x61

(input_string[0x18] + '\x05') = digit '6'
54 = input_string[0x18] + 5
input_string[0x18] = 49 = digit '1' in ASCII

('\b' - input_string[0x17] / '\x02') = digit '1'
49 = 8 - input_string[0x17] / 2
41 = - input_string[0x17] / 2
input_string[0x17] = -82 which in signed is 174 in 2s complement
input_string[0x17] = ® in ASCII (174 decimal)

it can also be -83 because of integer division which is 173 which is 'SHY' in ASCII soft hyphen
```


```
local_38 = calc_four_func((int)(char)(input_string[0x16] + -0xf ));
  local_37 = calc_four_func((int)(char)(input_string[0x15] + -0x3d));
  local_36 = 0;
  lVar2 = strtol(&local_38,(char **)0x0,0x10);
  local_20 = (int)lVar2;
(local_20 == 0x29)

using above logic, local_38 is digit '2' and local_37 is digit '9'

(input_string[0x16] + -0xf ) = digit '2'
50 = input_string[0x16] - 15
input_string[0x16] = 65 which is 'A' in ASCII

(input_string[0x15] + -0x3d) = digit '9'
57 = input_string[0x15] - 61
input_string[0x15] = 118 which is 'v' in ASCII
```

---------------------

from `calc_one` function

We can use the last 'if' statement because after that the string is not modified

```
if ((input_buff[0x14] == -0x4b) && (input_buff[0xd] == 's')) {
            flag_func(input_buff);}

(input_buff[0x14] == -0x4b)
input_buff[0x14] = -75 which is 181 in 2s complement
181 is 'µ' in ASCII

(input_buff[0xd] == 's')
```

 Now since we have most of the characters in the function, we will start reversing using a python script and we will go step by step



###### Reversing methodology:-

<img width="1419" height="1092" alt="image" src="https://github.com/user-attachments/assets/55e6316a-f123-42b9-a55b-152351e1301a" />


```
def shift_left(inp_str,offset):
    copy_str = inp_str
    mod_str = ''
    for i in range(len(inp_str)):
        mod_str += copy_str[(i + offset) % len(inp_str)]
    return mod_str
# shift_left function shifts the characters in string by offset amount to the left

def inc_asci_new(inp_str, value):
    copy_str = list(inp_str)
    for i in range(value):
        copy_str[((i + value) % len(copy_str))] = chr((ord(copy_str[((i + value) % len(copy_str))]) + value) % 256)
    mod_str = ''.join(copy_str)
    #print(mod_str)
    return copy_str

# Above function is to increase ascii but in a more efficient way

def dec_asci_new(inp_str, value):
    copy_str = list(inp_str)
    for i in range(value):
        copy_str[(i * value) % len(copy_str)] = chr((ord(copy_str[(i * value) % len(copy_str)]) - value) % 256)
    mod_str = ''.join(copy_str)
    #print(mod_str)
    #print(copy_str)
    return copy_str

# Above function is to increase ascii but in a more efficient way

def even_rev(inp_str, param_2):
    string_one = shift_left(inp_str, param_2)
    list_two = dec_asci_new(string_one, param_2)
    #print(list_two)
    return list_two

def odd_rev(inp_str, param_2):
    list_one = inc_asci_new(inp_str, param_2)
    string_one = ''.join(list_one)
    string_two = shift_left(string_one, param_2)
    #print(list(string_two))
    return list(string_two)
```

All the functions are listed above

```
final_list = []
final_list.append('s') # index 0
final_list.append('e')
final_list.append('i')
final_list.append(chr(0x1e))
final_list.append('?')
final_list.append(chr(0x92)) # index 5 chr(0x92)    ’ is not used because of encoding mismatch
final_list.append('3')
final_list.append('^')
final_list.append(chr(0x7f))
final_list.append('`')
final_list.append('&') # index 10
final_list.append('«')
final_list.append('1')
final_list.append('s') # index 13
final_list.append('?')
final_list.append('?')
final_list.append('?')
final_list.append('?')
final_list.append('?')
final_list.append('?')
final_list.append('µ') # index 20
final_list.append('v')
final_list.append('A')
final_list.append('®')
final_list.append('1')
final_list.append('e')
final_list.append(chr(0xC0)) # index 26

final_str = ''.join(final_list)
```

This is one of the combination of the final strings we will be using, since some places can be input with more than one values

```
rev_seven_list = odd_rev(final_str, 7) # This is the first reverse in calc_one at line 15
print("Old rev seven list:")
print(rev_seven_list)
# ['e', '\x86', 'g', '-', '²', '8', 'z', '?', '?', '?', '?', '?', '?', 'µ', 'v', 'A', '®', '1', 'e', 'À', 's', 'e', 'i', '\x1e', '?', '\x92', '3']
# ascii of index 7,8,9,10,11,12,13 should be increased by 7 and then shift left by 7, which means older modified index 7 which
# is '^' gets converted to 'e' and is now at position index 0
# old index 13 's' gets converted to 'z' and is now at index 6

# in this rev_seven_list we have three conditions
# if ((input_buff[8] == 'Y') && ((input_buff[0xb] == 'Y' && (( byte)(input_buff[0xc] + 0x8cU) < 4))))
# index 8 and index 11 must be 'Y'
# ( byte)(input_buff[0xc] + 0x8cU) < 4
# it means chr present at 0xc which is index 12, you add 140 to it, if the sum exceeds 255, we have to do %256 to bring it down
# When you do this, your value must be one of {0,1,2,3}. Only letters satisfying this are decimal 116,117,118,119 which are t,u,v,w

# for now we will do index 8 and index 11 must be 'Y' and index 12 is 't' for now

rev_seven_list[8] = 'Y'
rev_seven_list[11] = 'Y'
rev_seven_list[12] = 't'

print("Rev seven list after adding constrainsts:")
print(rev_seven_list)
# ['e', '\x86', 'g', '-', '²', '8', 'z', '?', 'Y', '?', '?', 'Y', 't', 'µ', 'v', 'A', '®', '1', 'e', 'À', 's', 'e', 'i', '\x1e', '?', '\x92', '3']

```

```

# after this we will do rev 10 which is even rev present at line 13 in calc_one function
# We will first shift left by 10 and then decrease ascii by 10 for following new indices [0, 3, 6, 9, 10, 13, 16, 20, 23, 26]

# We will look at the output by shifting left by 10 first

rev_seven_string = ''.join(rev_seven_list)
shift_ten_string = shift_left(rev_seven_string, 10)
print("Shift left 10")
print(list(shift_ten_string))
# ['?', 'Y', 't', 'µ', 'v', 'A', '®', '1', 'e', 'À', 's', 'e', 'i', '\x1e', '?', '\x92', '3', 'e', '\x86', 'g', '-', '²', '8', 'z', '?', 'Y', '?']
# When we will check conditions if ((*input_buff == -0x65) && ((byte)(input_buff[0x1a] + 0x8dU) < 5))
# we will be checking the first and last characters only, after shifting left we observe that they are '?' which were placeholder values
# we will just substitute those values after using the dec_ascii on this shifted string, the '?' at first and last indices will have changed values because of dec_ascii
# as for the other '?' they are present at index 14 and index 24, so they won;t be affected at all

# doing the full rev_10 transform now

rev_ten_list= even_rev(rev_seven_string, 10)
print("Rev 10 string after shift and dec asci")
print(rev_ten_list)
# ['5', 'Y', 't', '«', 'v', 'A', '¤', '1', 'e', '¶', 'i', 'e', 'i', '\x14', '?', '\x92', ')', 'e', '\x86', 'g', '#', '²', '8', 'p', '?', 'Y', '5']
# As we can see '?' at first and last positions have been changed to '5' 

# we will now apply conditions ((*input_buff == -0x65) && ((byte)(input_buff[0x1a] + 0x8dU) < 5))
# index 0 must be -0x65 which in 2s complement is 0x9B, if we check the ASCII table online this corresponds to '›' BUT 
# if you try ord() of this you will get a very large value, this is because python follows a new encoding system

# THIS MEANS WE HAVE TO REPLACE THE CHARACTERS WE EARLIER WROTE AS WELL WITH THE ACTUAL HEX AND NOT THE CHARACTER DISPLAYED
# THE CHARACTERS HIGHLIGHTED IN BLUE IN EXTENDED ASCII TABLE ARE THE OLD ENCODING ONES, REPLACE THOSE WITH JUST THE HEX VALUES

#From the original string we needed to replace index 5 this way

# (byte)(input_buff[0x1a] + 0x8dU) < 5) 
# this condition means that the last character must be one of s,t,u,v,w

#replacing the first and last characters 

rev_ten_list[0] = chr(0x9B)
rev_ten_list[26] = 'w'

print("Modded rev_ten_list")
print(rev_ten_list)
# ['\x9b', 'Y', 't', '«', 'v', 'A', '¤', '1', 'e', '¶', 'i', 'e', 'i', '\x14', '?', '\x92', ')', 'e', '\x86', 'g', '#', '²', '8', 'p', '?', 'Y', 'w']

```

```
# Now for rev 24 even in line 11 of calc_one, we will first shift left and then dec ascii

#output after shifting left by 24
rev_ten_string = ''.join(rev_ten_list)

shift_twentyfour_string = shift_left(rev_ten_string, 24)
print("Shift left 24")
print(list(shift_twentyfour_string))

# ['?', 'Y', 'w', '\x9b', 'Y', 't', '«', 'v', 'A', '¤', '1', 'e', '¶', 'i', 'e', 'i', '\x14', '?', '\x92', ')', 'e', '\x86', 'g', '#', '²', '8', 'p']

#For decreasing ascii, the indices affected will be [0, 0, 0, 3, 3, 6, 6, 9, 9, 12, 12, 12, 15, 15, 15, 18, 18, 18, 21, 21, 21, 24, 24, 24]
# The '?' at index 0 will be affected now but not the one present at index 17

#Output after shift left 24 and dec ascii

rev_twentyfour_list = even_rev(rev_ten_string, 24)
print("Not modified rev 24 list")
print(rev_twentyfour_list)

# ['÷', 'Y', 'w', 'k', 'Y', 't', '{', 'v', 'A', 't', '1', 'e', 'n', 'i', 'e', '!', '\x14', '?', 'J', ')', 'e', '>', 'g', '#', 'j', '8', 'p']
# Our '?' at index 0 got converted to '÷' keep track of that
#Many of the characters have been transformed now

#Now we have to perform a rev three at line 10 in calc_one
# That will consist of increasing ascii and then shift left
# affected indices will be 3,4,5 in the above list printed and after when we shift left by 3, our '?' will come at index 14 which is the next condition

rev_twentyfour_string = ''.join(rev_twentyfour_list)

rev_three_list = odd_rev(rev_twentyfour_string, 3)
print("Not modified rev 3 list")
print(rev_three_list)

# ['n', '\\', 'w', '{', 'v', 'A', 't', '1', 'e', 'n', 'i', 'e', '!', '\x14', '?', 'J', ')', 'e', '>', 'g', '#', 'j', '8', 'p', '÷', 'Y', 'w']

# Our '÷' has gone to index 24, keep note and '?' has arrived at index 14
# condition if (input_buff[0xe] == 'R'), index 14 must be 'R'

rev_three_list[14] = 'R'
print("Modified rev three list")
print(rev_three_list)

# ['n', '\\', 'w', '{', 'v', 'A', 't', '1', 'e', 'n', 'i', 'e', '!', '\x14', 'R', 'J', ')', 'e', '>', 'g', '#', 'j', '8', 'p', '÷', 'Y', 'w']

```

```
# Now we need to perform rev 13 in line 8 in calc_one
# If everything goes right, the last condition will replace the only placeholder valure remaining which is currently '÷' at index 24
# in rev 13, first ascii will be increased and then it will be shifted left by 13
# After shifting left by 13, our index 24 will become index 11 and that is exactly the place for next condition!

# performing rev 13

rev_three_string = ''.join(rev_three_list)

rev_thirteen_list = odd_rev(rev_three_string, 13)
print("Unmodified rev 13 list")
print(rev_thirteen_list)

# ['!', '_', 'W', '6', 'r', 'K', 't', '0', 'w', 'E', '}', '\x04', 'f', 'w', 'n', '\\', 'w', '{', 'v', 'A', 't', '1', 'e', 'n', 'i', 'e', '!']

# Our '÷' got converted to '\x04', we will now replace it with the next condition
# if (input_buff[0xb] == 'o')
# index 11 must be 'o'

rev_thirteen_list[11] = 'o'
print("Modified rev 13 list")
print(rev_thirteen_list)
# ['!', '_', 'W', '6', 'r', 'K', 't', '0', 'w', 'E', '}', 'o', 'f', 'w', 'n', '\\', 'w', '{', 'v', 'A', 't', '1', 'e', 'n', 'i', 'e', '!']

```

```
# Now all our conditions are satisfied, we just have to reverse two times now to get the original string

# rev 6 at line 6 in calc_one

rev_thirteen_string = ''.join(rev_thirteen_list)

rev_six_list = even_rev(rev_thirteen_string, 6)
rev_six_string = ''.join(rev_six_list)

# rev 5 at line 5 in calc_one

rev_five_list = odd_rev(rev_six_string, 5)
print("Rev 5 list")
print(rev_five_list)
# ['t', 'e', '|', 's', 'a', 'w', '{', 'p', 'A', 't', '1', 'e', 'n', 'c', 'e', '!', '!', '_', 'W', '0', 'r', 'K', 'n', '0', 'w', '?', '}']
# We know that our flag is 'texsaw{}' in this format, it means we can replace '|' with x

orignal_string = ''.join(rev_five_list)
print('original string')
print(orignal_string)

# te|saw{pAt1ence!!_W0rKn0w?}

# This is the string we got and it works fine
# We also got texsaw{pAt1ence!!_W0rKn0w?} which also works fine
```


##### Making a KeyGen.py :-

Now since we know the logic behind reversing and we also got two strings which satisfy our binary, we can make a keygen which will make all possible strings that can satisfy our binary constraints

```
# This File will be used as a keygen for the switcheroo problem as there are multiple solutions to the question

import itertools

def inc_asci_new(inp_str, value): # Increases ascii by 'value' for specific indices of string
    copy_str = inp_str
    for i in range(value):
        copy_str[((i + value) % len(copy_str))] = chr((ord(copy_str[((i + value) % len(copy_str))]) + value) % 256)
    return copy_str

def dec_asci_new(inp_str, value): # Decreases ascii by 'value' for specific indices of string
    copy_str = inp_str
    for i in range(value):
        copy_str[(i * value) % len(copy_str)] = chr((ord(copy_str[(i * value) % len(copy_str)]) - value) % 256)
    return copy_str

def shift_left(inp_list, offset): # Function to shift characters in list by offset value
    copy_list = inp_list
    mod_list = []
    for i in range(len(copy_list)):
        mod_list.append(copy_list[(i + offset) % len(copy_list)])
    return mod_list

def even_rev(inp_list, param_2): # Function to reverse when param_2 is even
    string_one = shift_left(inp_list, param_2)
    list_two = dec_asci_new(string_one, param_2)
    return list_two

def odd_rev(inp_list, param_2): # Function to reverse when param_2 is odd
    list_one = inc_asci_new(inp_list, param_2)
    list_two = shift_left(list_one, param_2)
    return list_two

def full_transform(inp_list): #This function reverses our final string to give back our original
    rev_seven_list = odd_rev(inp_list, 7)
    rev_ten_list = even_rev(rev_seven_list, 10)
    rev_twentyfour_list = even_rev(rev_ten_list, 24)
    rev_three_list = odd_rev(rev_twentyfour_list, 3)
    rev_thirteen_list = odd_rev(rev_three_list, 13)
    rev_six_list = even_rev(rev_thirteen_list, 6)
    rev_five_list = odd_rev(rev_six_list, 5)
    print("".join(rev_five_list))
    return

# options is a list of lists to make all possible combinations of the final_string which is acceptable, total 80 combinations

options = [
    ['s'],
    ['e'],
    ['i'],
    [chr(0x1e)],
    ['R'],
    [chr(0x91), chr(0x92)],
    ['3'],
    ['^'],
    [chr(0x7f)],
    ['`'],
    ['&'],
    ['«'],
    ['1'],
    ['s'],
    ['ª'],
    ['Y'],
    ['}', '~', chr(0x7f), chr(0x80), chr(0x81)],
    ['¥'],
    ['Y'],
    ['t', 'u', 'v', 'w'],
    ['µ'],
    ['v'],
    ['A'],
    [chr(0xad), '®'],
    ['1'],
    ['e'],
    [chr(0xC0)]
]

combinations = list(itertools.product(*options)) # All 80 combinations present

print(f"Total combinations found: {len(combinations)}")

for combo in combinations:
    full_transform(list(combo))
```
Here is the keygen.py
```
# This File will be used as a keygen for the switcheroo problem as there are multiple solutions to the question

import itertools

def inc_asci_new(inp_str, value): # Increases ascii by 'value' for specific indices of string
    copy_str = inp_str
    for i in range(value):
        copy_str[((i + value) % len(copy_str))] = chr((ord(copy_str[((i + value) % len(copy_str))]) + value) % 256)
    return copy_str

def dec_asci_new(inp_str, value): # Decreases ascii by 'value' for specific indices of string
    copy_str = inp_str
    for i in range(value):
        copy_str[(i * value) % len(copy_str)] = chr((ord(copy_str[(i * value) % len(copy_str)]) - value) % 256)
    return copy_str

def shift_left(inp_list, offset): # Function to shift characters in list by offset value
    copy_list = inp_list
    mod_list = []
    for i in range(len(copy_list)):
        mod_list.append(copy_list[(i + offset) % len(copy_list)])
    return mod_list

def even_rev(inp_list, param_2): # Function to reverse when param_2 is even
    string_one = shift_left(inp_list, param_2)
    list_two = dec_asci_new(string_one, param_2)
    return list_two

def odd_rev(inp_list, param_2): # Function to reverse when param_2 is odd
    list_one = inc_asci_new(inp_list, param_2)
    list_two = shift_left(list_one, param_2)
    return list_two

def full_transform(inp_list): #This function reverses our final string to give back our original
    rev_seven_list = odd_rev(inp_list, 7)
    rev_ten_list = even_rev(rev_seven_list, 10)
    rev_twentyfour_list = even_rev(rev_ten_list, 24)
    rev_three_list = odd_rev(rev_twentyfour_list, 3)
    rev_thirteen_list = odd_rev(rev_three_list, 13)
    rev_six_list = even_rev(rev_thirteen_list, 6)
    rev_five_list = odd_rev(rev_six_list, 5)
    print("".join(rev_five_list))
    return

# options is a list of lists to make all possible combinations of the final_string which is acceptable, total 80 combinations

options = [
    ['s'],
    ['e'],
    ['i'],
    [chr(0x1e)],
    ['R'],
    [chr(0x91), chr(0x92)],
    ['3'],
    ['^'],
    [chr(0x7f)],
    ['`'],
    ['&'],
    ['«'],
    ['1'],
    ['s'],
    ['ª'],
    ['Y'],
    ['}', '~', chr(0x7f), chr(0x80), chr(0x81)],
    ['¥'],
    ['Y'],
    ['t', 'u', 'v', 'w'],
    ['µ'],
    ['v'],
    ['A'],
    [chr(0xad), '®'],
    ['1'],
    ['e'],
    [chr(0xC0)]
]

combinations = list(itertools.product(*options)) # All 80 combinations present

print(f"Total combinations found: {len(combinations)}")

for combo in combinations:
    full_transform(list(combo))
```


This file will give us 80 strings which will all work with the given binary:-

```
Total combinations found: 80
texsaw{pAs1ence!!_V0rKn0w?}
texsaw{pAt1ence!!_V0rKn0w?}
texsax{pAs1ence!!_V0rKn0w?}
texsax{pAt1ence!!_V0rKn0w?}
texsay{pAs1ence!!_V0rKn0w?}
texsay{pAt1ence!!_V0rKn0w?}
texsaz{pAs1ence!!_V0rKn0w?}
texsaz{pAt1ence!!_V0rKn0w?}
teysaw{pAs1ence!!_V0rKn0w?}
teysaw{pAt1ence!!_V0rKn0w?}
teysax{pAs1ence!!_V0rKn0w?}
teysax{pAt1ence!!_V0rKn0w?}
teysay{pAs1ence!!_V0rKn0w?}
teysay{pAt1ence!!_V0rKn0w?}
teysaz{pAs1ence!!_V0rKn0w?}
teysaz{pAt1ence!!_V0rKn0w?}
tezsaw{pAs1ence!!_V0rKn0w?}
tezsaw{pAt1ence!!_V0rKn0w?}
tezsax{pAs1ence!!_V0rKn0w?}
tezsax{pAt1ence!!_V0rKn0w?}
tezsay{pAs1ence!!_V0rKn0w?}
tezsay{pAt1ence!!_V0rKn0w?}
tezsaz{pAs1ence!!_V0rKn0w?}
tezsaz{pAt1ence!!_V0rKn0w?}
te{saw{pAs1ence!!_V0rKn0w?}
te{saw{pAt1ence!!_V0rKn0w?}
te{sax{pAs1ence!!_V0rKn0w?}
te{sax{pAt1ence!!_V0rKn0w?}
te{say{pAs1ence!!_V0rKn0w?}
te{say{pAt1ence!!_V0rKn0w?}
te{saz{pAs1ence!!_V0rKn0w?}
te{saz{pAt1ence!!_V0rKn0w?}
te|saw{pAs1ence!!_V0rKn0w?}
te|saw{pAt1ence!!_V0rKn0w?}
te|sax{pAs1ence!!_V0rKn0w?}
te|sax{pAt1ence!!_V0rKn0w?}
te|say{pAs1ence!!_V0rKn0w?}
te|say{pAt1ence!!_V0rKn0w?}
te|saz{pAs1ence!!_V0rKn0w?}
te|saz{pAt1ence!!_V0rKn0w?}
texsaw{pAs1ence!!_W0rKn0w?}
texsaw{pAt1ence!!_W0rKn0w?}
texsax{pAs1ence!!_W0rKn0w?}
texsax{pAt1ence!!_W0rKn0w?}
texsay{pAs1ence!!_W0rKn0w?}
texsay{pAt1ence!!_W0rKn0w?}
texsaz{pAs1ence!!_W0rKn0w?}
texsaz{pAt1ence!!_W0rKn0w?}
teysaw{pAs1ence!!_W0rKn0w?}
teysaw{pAt1ence!!_W0rKn0w?}
teysax{pAs1ence!!_W0rKn0w?}
teysax{pAt1ence!!_W0rKn0w?}
teysay{pAs1ence!!_W0rKn0w?}
teysay{pAt1ence!!_W0rKn0w?}
teysaz{pAs1ence!!_W0rKn0w?}
teysaz{pAt1ence!!_W0rKn0w?}
tezsaw{pAs1ence!!_W0rKn0w?}
tezsaw{pAt1ence!!_W0rKn0w?}
tezsax{pAs1ence!!_W0rKn0w?}
tezsax{pAt1ence!!_W0rKn0w?}
tezsay{pAs1ence!!_W0rKn0w?}
tezsay{pAt1ence!!_W0rKn0w?}
tezsaz{pAs1ence!!_W0rKn0w?}
tezsaz{pAt1ence!!_W0rKn0w?}
te{saw{pAs1ence!!_W0rKn0w?}
te{saw{pAt1ence!!_W0rKn0w?}
te{sax{pAs1ence!!_W0rKn0w?}
te{sax{pAt1ence!!_W0rKn0w?}
te{say{pAs1ence!!_W0rKn0w?}
te{say{pAt1ence!!_W0rKn0w?}
te{saz{pAs1ence!!_W0rKn0w?}
te{saz{pAt1ence!!_W0rKn0w?}
te|saw{pAs1ence!!_W0rKn0w?}
te|saw{pAt1ence!!_W0rKn0w?}
te|sax{pAs1ence!!_W0rKn0w?}
te|sax{pAt1ence!!_W0rKn0w?}
te|say{pAs1ence!!_W0rKn0w?}
te|say{pAt1ence!!_W0rKn0w?}
te|saz{pAs1ence!!_W0rKn0w?}
te|saz{pAt1ence!!_W0rKn0w?}
```
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)




