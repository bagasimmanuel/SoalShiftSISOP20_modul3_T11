# SoalShiftSISOP20_modul3_T11
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
