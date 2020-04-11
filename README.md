# SoalShiftSISOP20_modul3_T11
## No 1
### Lho kok gaada

## No 2
### Lho kok gaada

## No 3
 ```
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <ctype.h>
#include <dirent.h>
#include <pthread.h>
#include <errno.h>
  ```
  Beberapa Library yang digunakan pada soal Ini

```
char *getFileName(char *fName, char buff[]) {
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buff, "%s", token);
    token = strtok(NULL, "/");
  }
}
```
Mendeklarasikan fungsi untuk mendapatkan nama 

```
char *getExtension(char *fName, char buff[]) {
  char buffFileName[1337];
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buffFileName, "%s", token);
    token = strtok(NULL, "/");
  }
  int count = 0;
  token = strtok(buffFileName, ".");
  while(token != NULL) {
    count++;
    sprintf(buff, "%s", token);
    token = strtok(NULL, ".");
  }
  if (count <= 1) {
    strcpy(buff, "unknown");
  }

  return buff;
}
```
Mendeklarasikan fungsi untuk mendapatkan extention 

```
void dirChecking(char buff[]) {
  DIR *dr = opendir(buff);
  if (ENOENT == errno) {
    mkdir(buff, 0775);
    closedir(dr);
  }
}
```
mendeklarasikan fungsi untuk mengecek direktori, jika belum ada maka buat direktori

```
struct args {
  char *buffer;
};
```
Tipe data yang akan di passing

```
void *routine(void* arg) {
  char buffExt[100];
  char buffFileName[1337];
  char buffFrom[1337];
  char buffTo[1337];
  char cwd[1337];
  getcwd(cwd, sizeof(cwd));
  strcpy(buffFrom, (char *) arg);
```
mendeklarasikan fungsi routine dan mengambil cwd

```
  if (access(buffFrom, F_OK) == -1) {
    printf("File %s tidak ada\n", buffFrom);
    pthread_exit(0);
  }
  DIR* dir = opendir(buffFrom);
  if (dir) {
    printf("file %s berupa folder\n", buffFrom);
    pthread_exit(0);
  }
  closedir(dir);
```
jika file tidak ada maka akan menampilkan file tidak ada dan jika berupa direktori maka akan berisi file berupa direktori

```
  getFileName(buffFrom, buffFileName);
  strcpy(buffFrom, (char *) arg);

  getExtension(buffFrom, buffExt);
  for (int i = 0; i < sizeof(buffExt); i++) {
    buffExt[i] = tolower(buffExt[i]);
  }
  strcpy(buffFrom, (char *) arg);

  dirChecking(buffExt);
```
menjalankan fungsi getfilename dan get extention serta dirchecking

```
  sprintf(buffTo, "%s/%s/%s", cwd, buffExt, buffFileName);
  rename(buffFrom, buffTo);

  pthread_exit(0);
```
memindahkan isi cwd buffext dan buff name untuk menjadi buffTo, lalu memindahkan dari file asal ke tempat tujuan menggunakan rename

```
int main(int argc, char *argv[]) {
  if (argc == 1) {
    printf("Argument kurang\n");
    exit(1);
  }
  if (strcmp(argv[1], "-f") != 0 && strcmp(argv[1], "*") != 0 && strcmp(argv[1], "-d")) {
    printf("Argument tidak ada\n");
    exit(1);
```
jika argumen yang dimasukan cuma 1 maka menampilkan argumen salah, jka selain -f -d dan * maka akan mengeluarkan argumen tidak ada

```
  if (strcmp(argv[1], "-f") == 0) {
    if (argc <= 2) {
      printf("Argument salah\n");
      exit(1);
    }
```
jika menggunakan argumen -f namun kurang dari sama dengan 2 maka menampilkan argumen salah

```
    pthread_t tid[argc-2];
    for (int i = 2; i < argc; i++) {
      pthread_create(&tid[i-2], NULL, &routine, (void *)argv[i]);
    }
    for (int i = 2; i < argc; i++) {
      pthread_join(tid[i-2], NULL);
    }
    exit(0);
  }
```
Menjalankan fungsu routine untun argumen ke 2 dan seterusnya menggunakan pyhread create dan pthread join

```
  char *directory;
  if (strcmp(argv[1], "*") == 0) {
    if (argc != 2) {
      printf("Argument salah\n");
      exit(1);
    }
    char buff[1337];
    getcwd(buff, sizeof(buff));
    directory = buff;
  }
```
jika menggunakan argumen * namun argumen tidak sama dengan 2 maka menampilkan argumen salah

```
  if (strcmp(argv[1], "-d") == 0) {
    if (argc != 3) {
      printf("Argument salah\n");
      exit(1);
    }
    DIR* dir = opendir(argv[2]);
    if (dir) {
      directory = argv[2];
    } else if (ENOENT == errno) {
      printf("Directory tidak ada\n");
      exit(1);
    }
    closedir(dir);
  }
```
jika argumen tidak sama dengan tidak, mengeluarkan argumen salah. jika file tidak ada maka menampilkan direktori tidak ada

```
  int file_count = 0;
  DIR* dir = opendir(directory);
  struct dirent *entry;
  while ((entry = readdir(dir)) != NULL) {
    if (entry->d_type == DT_REG) {
      file_count++;
    }
  }
  closedir(dir);

  pthread_t tid[file_count];
  char buff[file_count][1337];
  int iter = 0;

  dir = opendir(directory);
  while ((entry = readdir(dir)) != NULL) {
    if (entry->d_type == DT_REG) {
      sprintf(buff[iter], "%s/%s", directory, entry->d_name);
      iter++;
    }
  }
  closedir(dir);
```
menghitung jumlah file 

```
  for (int i = 0; i < file_count; i++) {
    char  *test = (char*)buff[i];
    printf("%s\n", test);
    pthread_create(&tid[i], NULL, &routine, (void *)test);
  }

  for (int i = 0; i < file_count; i++) {
    pthread_join(tid[i], NULL);
  }

}
```
Untuk setiap file, dijalankan fungsi routine

## No 4
### 4a.c

  ```
  #include <stdio.h>
  #include <string.h>
  #include <pthread.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <sys/ipc.h>
  #include <sys/shm.h>
  #include <sys/types.h>
  #include <sys/wait.h>
  ```
  Beberapa Library yang digunakan pada soal Ini
  ```
  #define UNREADY -1
  #define READY 0
  #define TAKEN 1
  #define MAX 100
  ```
  Pendefinisian beberapa Konstanta sehingga enak dilihat

  ```
  struct shared{
      int status;
      int data[100];
  };
  ```
  Tipe data yang akan di passing

  ```
  int matA[4][2];
  int matB[2][5];
  int matC[4][5];
  ```
  Deklarasi Variabel Matrix

  ```
  int baris = 0;
  int row = 0;

  void* kali(void* arg) {
    if(row >= 5){  ///
      row = 0;
      baris++;
    }

    for(int i = 0; i < 2;i++){
      matC[baris][row] += matA[baris][i] * matB[i][row];
    }
    row++;
  }

  ```
  Digunakan untuk melakukan Perkalian Matrix, baris dan row merupakan global variabel

  ```
  srand(time(NULL));
  ```
  Untuk bisa menggunakan function rand();
  ```
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 2; j++) {
      matA[i][j] = rand()%19+1;
      printf("%3d", matA[i][j]);
    }
    printf("\n");
  }

  ```
  Mengisi Nilai Matrix dan melakukan Display

```
pthread_t tid[20];

for (int i = 0; i < 20; i++) {
  pthread_create(&(tid[i]), NULL, &kali, NULL);
}

for (int i = 0; i < 20; i++) {
  pthread_join(tid[i], NULL);
}
```
  Melakukan Perkalian Matrix menggunakan Thread, tidak lupa untuk join agar saling menunggu


```

   key_t          ShmKEY;
   int            ShmID;
   struct shared  *ShmPTR;

```
  ShmKey = SharedMemory Key
  ShmdID = SharedMemory ID
  ShmPtr = Struct yang akan di passing

```

   ShmKEY = ftok("key",100);
   ShmID = shmget(ShmKEY,sizeof(struct shared),IPC_CREAT|0666);
   if(ShmID < 0){
     printf("ERROR DI SHMGET!\n");
     exit(1);
   }
   ShmPTR = (struct shared *) shmat(ShmID, NULL, 0);

```
  Inisialisasi Proses Shared memory
  1. Membuat key dengan katakunci ("key",100)
  2. Membuat SharedMemoryID dengan key tersebut
  3. Mencantumkan Pointer dari struct yang akan di share ke ShmID


  ```
  ShmPTR->status = UNREADY;
  int j = 0;
  int k = 0;

  for(int i = 0; i < 20; i++){
    if(k >= 5){
      j++;
      k = 0;
    }
    ShmPTR->data[i] = matC[j][k];
    k++;
  }
  ```
  Memasukkan Data Matrix ke dalam Struct

  ```
  printf("Please start the client in another window...\n");
  while (ShmPTR->status != TAKEN)
      sleep(1);
  ```
  Menunggu hingga soal 4b.c dijalankan untuk menerima data yang telah dipassing

  ```
  printf("Server has detected the completion of its child...\n");
   shmdt((void *) ShmPTR);
   printf("Server has detached its shared memory...\n");
   shmctl(ShmID, IPC_RMID, NULL);
   printf("Server has removed its shared memory...\n");
   printf("Server exits...\n");
   exit(0);

  ```
  Jika sudah Diterima maka akan di destroy

  Source Code : [4a.c](/no4/4a.c)


### 4b

```
void* sumOf(void* arg) {
  int i = *((int*)arg);
  free(arg);;
  int total = 0;
  for(int j = 0; j <= i ;j++){
    total += j;
  }
  if(row > 4){
    printf("\n");
    row = 0;
  }
  printf("%d %10d ",i,total);
  row ++;
}

```
  Fungsi yang akan digunakan di thread untuk mendapatkan jumlah dari 0 hingga n

```

     key_t          ShmKEY;
     int            ShmID;
     struct shared  *ShmPTR;

     ShmKEY = ftok("key",100);
     ShmID = shmget(ShmKEY,sizeof(struct shared),0666);
     if(ShmID < 0){
       printf(" ** SHMERROR CLIENT! ** \n");
       exit(1);
     }
     ShmPTR = (struct shared*) shmat(ShmID, NULL, 0);

```

  Sama seperti 4a. Melakukan Inisialisasi Shared Memory

  ```

  while (ShmPTR->status != READY)
     ;

  ```
  Selama 4a belum melakukan passing data, maka akan menggungu 4a

  ```

  pthread_t tid[20];
  for(int i = 0; i < 20;i++){
     int *x =  malloc(sizeof(*x));
     if( x == NULL){
       printf("ERROR CANT MAKE NEW SPACE\n");
       exit(1);
     }
     *x = ShmPTR->data[i];
     pthread_create(&(tid[i]), NULL, &sumOf, x);
    pthread_join(tid[i], NULL);

  }

  ```

  Setelah data diterima, maka dilakukan thread sebanyak jumlah data, dan didapatkan dan didisplay sumOf dari tiap tiap nilai yang didapatkan

  ```
  ShmPTR->status = TAKEN;
  shmdt((void *) ShmPTR);
  ```
  Setelah data diterima, maka status diubah menjadi taken yang mengindikasikan bahwa data telah diterima dan proses 4a boleh exit

  Source Code : [4b.c](/no4/4b.c)

  ### 4c

    ```
    int fd[2];
    pid_t pid;
    pipe(fd);

    ```
  Inisialisasi Pipe

  ```
  pid = fork();
  if (pid == 0) {
    dup2(fd[1], 1);
    close(fd[0]);
    close(fd[1]);
    char *argv[] = {"ls", NULL};
    execv("/bin/ls", argv);
  }
  while(wait(NULL) > 0);
  ```
  Pertama - tama lakukan fork untuk mendapatkan child process kemudian Jalankan execv bin/ls dan masukkan input ke dalam fd[1] dengan function dup2 dan juga argumen 1 untuk melakukan write


  ```
  pid = fork();
  if (pid == 0) {
    dup2(fd[0], 0);
    close(fd[0]);
    close(fd[1]);
    char *argv[] = {"wc", "-l", NULL};
    execv("/usr/bin/wc", argv);
  }
  ```
  Melakukan Fork lagi tetapi kali ini untuk membaca output dari fd[1], dan membacanya, dibaca dengan function dup2 dengan argumen (0) lalu lakukan execv "wc -l"

  # LINGLING 
