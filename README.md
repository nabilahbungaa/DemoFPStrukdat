# DemoFPStrukdat

Program ini yang merupakan final project mata kuliah struktur data kelas B adalah aplikasi pengelolaan data pegawai menggunakan file biner, dengan fitur:
Tambah/Ubah/Hapus data pegawai
Menyimpan data di pegawai.dat
Menyimpan indeks ID dan posisi file di indeks.dat untuk mempercepat pencarian
Tampilkan data pakai indeks atau tidak
Simpan data dengan struct dan kontrol input lengkap

###  BAGIAN 1: HEADER DAN DEFINISI AWAL
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <conio.h>
#include <ctype.h>
```
Tujuan: Mengimpor library standar C untuk input/output (stdio), alokasi memori (stdlib), manipulasi string (string), input keyboard (conio) dan pengecekan karakter (ctype).
conio.h untuk getch() di Windows. Tapi karena gak semua OS punya conio.h, dibuat alternatif...

```
typedef struct {
    char id[7];        
    char nama[26];     
    int gender;       
    double gaji;     
    int is_deleted;    
} Pegawai;
```
Struktur Pegawai adalah objek utama yang disimpan di file.
id adalah ID pegawai (6 digit + \0)
nama maksimal 25 karakter + null terminator
gender = 0 (perempuan) atau 1 (laki-laki)
gaji bertipe double
is_deleted untuk penandaan data dihapus secara logis

```
typedef struct {
    char id[7];        
    long posisi;    
} Indeks;
```
Struktur Indeks menyimpan id dan posisi dalam file biner.
Untuk mempercepat pencarian via indeks

```
FILE *file_data;
Indeks daftar_indeks[1000];  
int jumlah_indeks = 0;
```
file_data menyimpan pointer ke file pegawai.dat
daftar_indeks[] menyimpan array dari semua indeks ID pegawai
jumlah_indeks menyimpan jumlah total data aktif

## FUNGSI FORMAT GAJI

```
void format_gaji(double gaji, char *buffer) {
```
Fungsi format_gaji() bertujuan untuk mengubah format gaji (bilangan desimal) menjadi format mata uang Indonesia dengan pemisah ribuan (titik) dan pemisah desimal (koma). Misalnya, 1000000.00 akan diformat menjadi 1.000.000,00.

```
char temp[20];
```
Di sini kita mendeklarasikan array temp dengan kapasitas 20 karakter untuk menyimpan representasi sementara dari angka gaji dalam format string.
Ukuran 20 cukup besar untuk menyimpan angka gaji beserta koma dan titik ribuan.

```
sprintf(temp, "%.2f", gaji);
```
Format %.2f memastikan bahwa gaji akan dibulatkan hingga dua angka setelah titik desimal.
Hasilnya disimpan di dalam array temp.
Misalnya, jika gaji = 1000000.5, maka temp akan berisi "1000000.50".

```
char *decimal_point = strchr(temp, '.');
```
Fungsi strchr() digunakan untuk mencari posisi karakter pertama yang cocok dengan titik desimal ('.') dalam string temp.
Jika ditemukan, decimal_point akan menunjuk ke lokasi titik desimal dalam string. Jika tidak ada titik desimal, decimal_point akan bernilai NULL.

```
if (decimal_point) *decimal_point = ',';
```
Kondisi ini memeriksa apakah titik desimal ('.') ditemukan di string temp.
Jika ditemukan (nilai decimal_point bukan NULL), kita mengganti titik ('.') tersebut dengan koma (,) agar sesuai dengan format mata uang Indonesia yang menggunakan koma sebagai pemisah desimal.

```
int len = strlen(temp);
```
Fungsi strlen() digunakan untuk menghitung panjang string temp setelah proses pemformatan.

```
int comma_pos = (decimal_point) ? decimal_point - temp : len;
```
Fungsi strlen() digunakan untuk menghitung panjang string temp setelah proses pemformatan.

```
int new_len = 0;
```
Variabel new_len digunakan untuk melacak posisi penulisan karakter dalam array buffer.

```
for (int i = 0; i < len; i++) {
    if (i > 0 && (comma_pos - i) % 3 == 0 && i < comma_pos) {
        buffer[new_len++] = '.';
    }
    buffer[new_len++] = temp[i];
}
```
```i > 0 && (comma_pos - i) % 3 == 0 && i < comma_pos:```
Cek apakah kita sudah melewati titik desimal (misalnya, 3 digit setelah koma), dan jika ya, tambahkan titik untuk pemisah ribuan.
```comma_pos - i``` menghitung jarak dari titik desimal ke karakter yang sedang diproses. Jika jarak tersebut kelipatan 3, artinya kita sudah berada pada tempat yang tepat untuk menyisipkan titik sebagai pemisah ribuan.

```buffer[new_len++] = '.';:``` Jika kondisi di atas terpenuhi, kita menambahkan titik (.) ke dalam buffer dan increment new_len.
```buffer[new_len++] = temp[i];:``` Setelah itu, salin karakter dari temp ke dalam buffer.

## Fungsi bandingkan_indeks

```
int bandingkan_indeks(const void *a, const void *b)
```
bandingkan_indeks adalah fungsi pembanding yang digunakan oleh fungsi qsort().
const void *a, const void *b: Ini adalah parameter generik untuk menerima dua elemen yang akan dibandingkan. void * digunakan karena qsort() bekerja dengan tipe data apapun, dan kita akan mengonversinya ke tipe yang sesuai di dalam fungsi ini.
Fungsi ini mengembalikan nilai int, yang berfungsi untuk menentukan urutan dua elemen yang dibandingkan:
Nilai negatif berarti a lebih kecil dari b
Nilai positif berarti a lebih besar dari b
Nilai nol berarti a dan b sama.\

```
return strcmp(((Indeks*)a)->id, ((Indeks*)b)->id);
```
(Indeks*)a: Kita mengonversi pointer a dari void* ke pointer Indeks*. Ini karena qsort() tidak tahu tipe data yang dibandingkan, sehingga kita perlu memberi tahu bahwa a adalah pointer ke objek bertipe Indeks.
((Indeks*)a)->id: Setelah mengonversi ke pointer Indeks*, kita mengakses anggota id dari objek Indeks yang ditunjuk oleh a. Ini adalah ID pegawai yang disimpan dalam tipe data char[7].
strcmp() adalah fungsi standar C yang membandingkan dua string (dalam hal ini, dua ID).
Fungsi ini mengembalikan:
Nilai negatif jika string pertama lebih kecil dari string kedua.
Nilai positif jika string pertama lebih besar dari string kedua.
Nilai nol jika kedua string tersebut identik.

## Fungsi bersihkan_buffer

```
void bersihkan_buffer()
```
Fungsi ini tidak menerima parameter dan tidak mengembalikan nilai.
Tujuan utama fungsi ini adalah untuk membersihkan buffer input, khususnya setelah menggunakan scanf() yang sering kali meninggalkan karakter newline (\n) di buffer.

```
while (getchar() != '\n');
```
Fungsi ini akan membuang semua karakter dalam buffer sampai menemukan karakter newline ('\n'), yang menandakan akhir dari input pengguna.

## FUNGSI IS NUMERIC

```
int is_numeric(const char *str)
```
Fungsi ini memeriksa apakah string str hanya terdiri dari angka, dan bisa mengandung satu titik atau koma sebagai pemisah desimal.
Mengembalikan 1 jika valid, 0 jika tidak.

```
int titik_ditemukan = 0;
```
Variabel titik_ditemukan digunakan untuk memeriksa apakah sudah ditemukan titik atau koma dalam string. Ini untuk memastikan hanya satu titik/koma yang diperbolehkan.

```
if (*str == '-') str++; 
```
Jika karakter pertama adalah tanda minus (-), maka kita lewati dan periksa karakter berikutnya (ini untuk mendukung angka negatif).

```
while (*str) {
    if (*str == '.' || *str == ',') {
        if (titik_ditemukan) return 0; 
        titik_ditemukan = 1;
    } else if (!isdigit(*str)) {
        return 0;
    }
    str++;
}
```
Loop ini memeriksa setiap karakter dalam string:
Jika ada titik atau koma, pastikan hanya satu yang ada (jika sudah ada, return 0).
Jika karakter bukan angka atau tanda desimal yang sah, return 0.
Jika karakter valid, lanjutkan ke karakter berikutnya.

```
return 1;
```
Jika tidak ada kesalahan ditemukan, berarti string valid, sehingga fungsi mengembalikan 1.

## FUNGSI INPUT GAJI

```
char input[20];
double gaji;
```
input adalah array untuk menyimpan input pengguna berupa string (maksimal 19 karakter + null terminator).
gaji adalah variabel yang akan menyimpan nilai gaji setelah diproses.

```
while (1) {
```
Loop tak terbatas digunakan untuk terus meminta input pengguna sampai input valid diterima.

```
printf("Gaji Pokok (Rp.): ");
scanf(" %19[^\n]", input);
bersihkan_buffer();
```
Menampilkan prompt untuk input gaji.
scanf(" %19[^\n]", input) membaca input hingga 19 karakter, memastikan tidak ada karakter lebih dari itu.
bersihkan_buffer() digunakan untuk menghapus karakter newline yang tertinggal setelah input.

```
for (char *p = input; *p; p++) {
    if (*p == ',') *p = '.';
}
```
Loop ini mengganti semua koma , dengan titik . untuk memastikan format angka sesuai dengan desimal yang benar (untuk gaji).

```
char cleaned[20];
int j = 0;
for (int i = 0; input[i] && j < 19; i++) {
    if (isdigit(input[i]) || input[i] == '.' || input[i] == ',') {
        cleaned[j++] = input[i];
    }
}
cleaned[j] = '\0';
```
Membuat string cleaned untuk menyaring hanya karakter yang valid: angka, titik, atau koma.
Setiap karakter yang valid dimasukkan ke cleaned[], dan akhirnya ditambahkan \0 untuk mengakhiri string.

```
if (!is_numeric(cleaned)) {
    printf("Format gaji tidak valid! Harus angka (contoh: 5.000.000 atau 5,000,000)\n");
    continue;
}
```
Memanggil fungsi is_numeric() untuk memeriksa apakah format gaji valid (hanya angka, satu titik/koma).
Jika tidak valid, menampilkan pesan kesalahan dan melanjutkan loop untuk meminta input ulang.

```
gaji = atof(cleaned);
```
Mengonversi string cleaned menjadi angka double yang disimpan di variabel gaji

```
if (gaji <= 0) {
    printf("Gaji harus lebih dari 0!\n");
    continue;
}

if (gaji > 1000000000) {
    printf("Gaji maksimal Rp. 1.000.000.000!\n");
    continue;
}
```
Memeriksa apakah gaji lebih besar dari 0 dan tidak melebihi Rp. 1.000.000.000.
Jika tidak memenuhi syarat, menampilkan pesan kesalahan dan meminta input ulang.

```
break;
```
Jika input valid, keluar dari loop dan lanjutkan ke baris selanjutnya.

```
return gaji;
```
Mengembalikan nilai gaji yang telah valid dan diproses.

## FUNGSI CARI DATA

```
int cari_data(char *id)
```
Fungsi ini digunakan untuk mencari data pegawai berdasarkan ID menggunakan metode binary search.
Fungsi ini mengembalikan indeks posisi data jika ditemukan, atau -1 jika tidak ditemukan.

```
int kiri = 0, kanan = jumlah_indeks - 1;
```
kiri dan kanan adalah indeks batas dari array daftar_indeks[] yang akan dicari. kiri dimulai dari indeks pertama (0) dan kanan dari indeks terakhir (jumlah_indeks - 1).

```
while (kiri <= kanan) {
    int tengah = (kiri + kanan) / 2;
    int hasil = strcmp(daftar_indeks[tengah].id, id);
    if (hasil == 0) return tengah;
    if (hasil < 0) kiri = tengah + 1;
    else kanan = tengah - 1;
}
```
```while (kiri <= kanan)``` Selama batas kiri tidak melebihi kanan, pencarian terus dilakukan.
```tengah = (kiri + kanan) / 2``` Menghitung indeks tengah antara kiri dan kanan.
```strcmp(daftar_indeks[tengah].id, id)``` Membandingkan ID pada indeks tengah dengan ID yang dicari.
Jika ```hasil == 0```berarti ID ditemukan, dan fungsi mengembalikan indeks tengah.
Jika ```hasil < 0```, ID yang dicari lebih besar dari ID tengah, maka pergeseran dilakukan dengan mengubah kiri menjadi tengah + 1 untuk mencari di sebelah kanan.
Jika ```hasil > 0```, ID yang dicari lebih kecil dari ID tengah, maka pergeseran dilakukan dengan mengubah kanan menjadi tengah - 1 untuk mencari di sebelah kiri.\

```
return -1;
```
Jika pencarian selesai dan ID tidak ditemukan, fungsi mengembalikan -1.

## FUNGSI MUAT INDEKS

```
FILE *file_indeks = fopen("indeks.dat", "rb");
```
Membuka file indeks.dat untuk dibaca dalam mode binary ("rb").
file_indeks adalah pointer yang akan menunjuk ke file tersebut jika berhasil dibuka.

```
if (!file_indeks) return;
```
Jika file gagal dibuka (misalnya file tidak ada), maka fungsi langsung keluar tanpa melakukan apa-apa (return).

```
jumlah_indeks = 0;
```
Menginisialisasi variabel jumlah_indeks dengan 0, yang akan menghitung jumlah elemen yang berhasil dibaca dari file.

```
while (fread(&daftar_indeks[jumlah_indeks], sizeof(Indeks), 1, file_indeks)) {
    jumlah_indeks++;
}
```
fread() membaca data dari file file_indeks dan menyimpannya ke dalam array daftar_indeks[].
fread(&daftar_indeks[jumlah_indeks], sizeof(Indeks), 1, file_indeks) membaca satu elemen data bertipe Indeks ke dalam array daftar_indeks pada posisi jumlah_indeks.```
Setiap kali data berhasil dibaca, jumlah_indeks ditambah 1.

```
fclose(file_indeks);
```
Setelah selesai membaca semua data, file ditutup dengan fclose() untuk membebaskan sumber daya yang digunakan.

```
qsort(daftar_indeks, jumlah_indeks, sizeof(Indeks), bandingkan_indeks);
```
qsort() digunakan untuk mengurutkan array daftar_indeks[] berdasarkan ID dengan menggunakan fungsi pembanding bandingkan_indeks().
jumlah_indeks adalah jumlah elemen yang akan diurutkan, dan sizeof(Indeks) menentukan ukuran tiap elemen.

## FUNGSI SIMPAN INDEKS

```
FILE *file_indeks = fopen("indeks.dat", "wb");
```
Fungsi fopen() membuka file indeks.dat untuk menulis data dalam mode binary ("wb").
Jika file belum ada, file baru akan dibuat. Jika gagal membuka file, file_indeks akan bernilai NULL.

```
if (!file_indeks) return;
```
Jika file gagal dibuka (misalnya, karena hak akses atau kesalahan lainnya), maka fungsi akan langsung keluar tanpa melakukan apapun (return).

```
fwrite(daftar_indeks, sizeof(Indeks), jumlah_indeks, file_indeks);
```
Fungsi fwrite() digunakan untuk menulis data ke dalam file.
daftar_indeks: Array yang berisi data indeks yang ingin disimpan.
sizeof(Indeks): Ukuran masing-masing elemen dalam array (tipe Indeks).
jumlah_indeks: Jumlah elemen yang ingin ditulis dari array daftar_indeks ke dalam file.
file_indeks: Pointer file tempat data akan ditulis.

```
fclose(file_indeks);
```
Fungsi fclose() menutup file setelah selesai menulis data untuk membebaskan sumber daya yang digunakan.


## FUNGSI TAMBAH DATA

```Pegawai pegawai;```
Mendeklarasikan variabel pegawai yang bertipe Pegawai, yang akan digunakan untuk menyimpan data pegawai yang baru.

```
printf("\n=== TAMBAH DATA PEGAWAI ===\n");
```
Menampilkan pesan untuk memberi tahu pengguna bahwa mereka sedang memasukkan data pegawai baru.

```]
while (1) {
    printf("ID Pegawai (6 digit angka): ");
    scanf("%6s", pegawai.id);
    bersihkan_buffer();
    
    if (strlen(pegawai.id) != 6) {
        printf("ID harus 6 digit!\n");
        continue;
    }
    
    int valid = 1;
    for (int i = 0; i < 6; i++) {
        if (!isdigit(pegawai.id[i])) {
            valid = 0;
            break;
        }
    }
    
    if (!valid) {
        printf("ID harus angka semua!\n");
        continue;
    }
    
    if (cari_data(pegawai.id) != -1) {
        printf("Error: ID sudah ada!\n");
        continue;
    }
    
    break;
}
```
Meminta pengguna untuk memasukkan ID pegawai yang terdiri dari 6 digit angka.
Melakukan validasi:
ID harus berjumlah 6 digit.
Semua karakter dalam ID harus berupa angka.
ID tidak boleh duplikat, dengan memeriksa ID yang sudah ada menggunakan cari_data().

```
printf("Nama Pegawai (maks 25 huruf): ");
scanf(" %25[^\n]", pegawai.nama);
bersihkan_buffer();
```
Meminta pengguna untuk memasukkan nama pegawai, maksimal 25 karakter, dan menghapus karakter newline setelah input.

```
while (1) {
    printf("Gender (0: Perempuan, 1: Laki-laki): ");
    scanf("%d", &pegawai.gender);
    bersihkan_buffer();
    
    if (pegawai.gender != 0 && pegawai.gender != 1) {
        printf("Masukkan 0 atau 1!\n");
        continue;
    }
    break;
}
```
Meminta pengguna untuk memasukkan gender pegawai.
Validasi input agar hanya menerima 0 untuk perempuan atau 1 untuk laki-laki.

```
pegawai.gaji = input_gaji();
```
Memanggil fungsi input_gaji() untuk meminta dan memvalidasi gaji pegawai.

```
pegawai.is_deleted = 0;
```
Menetapkan nilai is_deleted ke 0, yang berarti data pegawai ini tidak dihapus (aktif).

```
fseek(file_data, 0, SEEK_END);
long posisi = ftell(file_data);
fwrite(&pegawai, sizeof(Pegawai), 1, file_data);
fflush(file_data);
```
fseek() memindahkan pointer file ke akhir file (SEEK_END).
ftell() mencatat posisi saat ini di file, yang akan digunakan untuk menyimpan lokasi data.
fwrite() menulis data pegawai ke dalam file pada posisi yang tercatat.  
fflush() memastikan data disimpan ke disk segera.

```
strcpy(daftar_indeks[jumlah_indeks].id, pegawai.id);
daftar_indeks[jumlah_indeks].posisi = posisi;
jumlah_indeks++;
```
Menyalin ID pegawai ke dalam array daftar_indeks[] pada posisi yang sesuai.
Menyimpan posisi data pegawai di file pada array daftar_indeks[].
Meningkatkan jumlah indeks yang sudah tercatat.

```
qsort(daftar_indeks, jumlah_indeks, sizeof(Indeks), bandingkan_indeks);
```
Mengurutkan array daftar_indeks[] berdasarkan ID menggunakan fungsi qsort() dan fungsi pembanding bandingkan_indeks().

```
simpan_indeks();
```
Memanggil fungsi simpan_indeks() untuk menyimpan array indeks yang telah diperbarui ke dalam file indeks.dat.

```
printf("\nData berhasil ditambahkan!\n");
```
Menampilkan pesan untuk memberi tahu pengguna bahwa data pegawai telah berhasil ditambahkan.

## FUNGSI UBAH DATA

```
char id[7];
```
Mendeklarasikan variabel id untuk menyimpan ID pegawai yang akan diubah, dengan panjang 6 karakter dan tambahan null-terminator ('\0'

```
printf("\n=== UBAH DATA PEGAWAI ===\n");
printf("Masukkan ID Pegawai yang akan diubah: ");
scanf("%6s", id);
bersihkan_buffer();
```
Menampilkan pesan untuk memberi tahu pengguna bahwa mereka akan mengubah data pegawai.
Meminta pengguna untuk memasukkan ID pegawai yang ingin diubah, kemudian membersihkan buffer input untuk menghindari karakter yang tidak diinginkan.

```
int indeks = cari_data(id);
if (indeks == -1) {
    printf("Error: ID tidak ditemukan!\n");
    return;
}
```
Menggunakan fungsi cari_data(id) untuk mencari ID pegawai di dalam array indeks.
Jika ID tidak ditemukan (indeks == -1), menampilkan pesan error dan keluar dari fungsi.



