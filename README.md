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

```
fseek(file_data, daftar_indeks[indeks].posisi, SEEK_SET);
Pegawai pegawai;
fread(&pegawai, sizeof(Pegawai), 1, file_data);
```
fseek(): Mengatur posisi file ke lokasi data pegawai yang ingin diubah, berdasarkan posisi yang disimpan di array daftar_indeks[].
fread(): Membaca data pegawai dari file ke dalam variabel pegawai.

```
printf("\nData Lama:\n");
printf("ID: %s\n", pegawai.id);
printf("Nama: %s\n", pegawai.nama);
printf("Gender: %s\n", pegawai.gender ? "Laki-laki" : "Perempuan");
```
Menampilkan data pegawai yang lama, seperti ID, nama, gender, dan gaji dengan format yang sesuai.

```
char formatted_gaji[20];
format_gaji(pegawai.gaji, formatted_gaji);
printf("Gaji: Rp. %s\n", formatted_gaji);
```
Menggunakan fungsi format_gaji() untuk memformat gaji agar tampil dengan format yang lebih mudah dibaca.
Menampilkan gaji yang sudah diformat.

```
printf("\nMasukkan Data Baru:\n");
```
Memberikan penanda bahwa sekarang pengguna akan memasukkan data baru untuk pegawai tersebut

```
printf("Nama Pegawai (maks 25 huruf): ");
scanf(" %25[^\n]", pegawai.nama);
bersihkan_buffer();
```
Meminta pengguna untuk memasukkan nama baru pegawai, dengan panjang maksimal 25 karakter.
Menggunakan bersihkan_buffer() untuk memastikan input berikutnya tidak terpengaruh oleh karakter yang tidak diinginkan.

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
Meminta pengguna untuk memasukkan gender baru dengan validasi:
Gender harus 0 untuk perempuan atau 1 untuk laki-laki.
Jika input tidak valid, pengguna diminta untuk mencoba lagi.

```
pegawai.gaji = input_gaji();
```
Memanggil fungsi input_gaji() untuk meminta dan memvalidasi gaji baru yang akan dimasukkan.

```
fseek(file_data, daftar_indeks[indeks].posisi, SEEK_SET);
fwrite(&pegawai, sizeof(Pegawai), 1, file_data);
fflush(file_data);
```
fseek(): Mengatur posisi file kembali ke posisi data pegawai yang akan diubah.
fwrite(): Menulis data pegawai yang sudah diperbarui ke file.
fflush(): Memastikan perubahan segera disimpan ke disk.

```
printf("\nData berhasil diubah!\n");
```
Menampilkan pesan yang menunjukkan bahwa data pegawai telah berhasil diubah.

## FUNGSI HAPUS DATA

```
char id[7];
```
Mendeklarasikan variabel id yang akan menyimpan ID pegawai yang akan dihapus.

```
printf("\n=== HAPUS DATA PEGAWAI ===\n");
printf("Masukkan ID Pegawai yang akan dihapus: ");
scanf("%6s", id);
bersihkan_buffer();
```
Menampilkan pesan untuk memberi tahu pengguna bahwa mereka akan menghapus data pegawai.
Meminta pengguna untuk memasukkan ID pegawai yang ingin dihapus, kemudian membersihkan buffer input.

```
int indeks = cari_data(id);
if (indeks == -1) {
    printf("Error: ID tidak ditemukan!\n");
    return;
}
```
Menggunakan fungsi cari_data(id) untuk mencari ID pegawai yang akan dihapus.
Jika ID tidak ditemukan (indeks == -1), menampilkan pesan error dan keluar dari fungsi.

```
fseek(file_data, daftar_indeks[indeks].posisi, SEEK_SET);
Pegawai pegawai;
fread(&pegawai, sizeof(Pegawai), 1, file_data);
```
fseek(): Mengatur posisi file ke lokasi data pegawai yang akan dihapus berdasarkan posisi yang disimpan dalam daftar_indeks[].
fread(): Membaca data pegawai dari file ke dalam variabel pegawai.

```
pegawai.is_deleted = 1;
```
Menandakan bahwa data pegawai ini dihapus dengan mengubah nilai is_deleted menjadi 1.

```
fseek(file_data, daftar_indeks[indeks].posisi, SEEK_SET);
fwrite(&pegawai, sizeof(Pegawai), 1, file_data);
fflush(file_data);
```
fseek(): Mengatur posisi file ke lokasi data pegawai yang akan dihapus berdasarkan posisi yang disimpan dalam daftar_indeks[].
fread(): Membaca data pegawai dari file ke dalam variabel pegawai.

```
pegawai.is_deleted = 1;
```
Menandakan bahwa data pegawai ini dihapus dengan mengubah nilai is_deleted menjadi 1.

```
fseek(file_data, daftar_indeks[indeks].posisi, SEEK_SET);
fwrite(&pegawai, sizeof(Pegawai), 1, file_data);
fflush(file_data);
```
fseek(): Memindahkan pointer file kembali ke posisi data pegawai yang akan diubah.
fwrite(): Menulis kembali data pegawai yang telah diubah (menandakan sebagai dihapus) ke file.
fflush(): Memastikan perubahan disimpan segera ke disk.

```
for (int i = indeks; i < jumlah_indeks - 1; i++) {
    daftar_indeks[i] = daftar_indeks[i + 1];
}
jumlah_indeks--;
```
Menggeser elemen-elemen dalam array daftar_indeks[] untuk menghapus indeks yang terkait dengan pegawai yang telah dihapus.
Mengurangi jumlah indeks yang tercatat (jumlah_indeks--).

```
simpan_indeks();
```
Memanggil fungsi simpan_indeks() untuk menyimpan array indeks yang telah diperbarui ke dalam file indeks.dat.

```
printf("\nData berhasil dihapus!\n");
```
Menampilkan pesan yang memberitahukan bahwa data pegawai telah berhasil dihapus.

## FUNGSI TAMPILKAN DATA

```
system("cls");
```
Fungsi system("cls") digunakan untuk membersihkan layar terminal sebelum menampilkan data.

```
printf("%s\n", pakai_indeks ? "DATA PEGAWAI (DENGAN INDEKS)" : "DATA PEGAWAI (TANPA INDEKS)");
```
Menampilkan judul "DATA PEGAWAI" dengan keterangan apakah data ditampilkan menggunakan indeks atau tidak, berdasarkan nilai pakai_indeks.

```
printf("+--------+---------------------------+------------+---------------------+\n");
printf("| %-6s | %-25s | %-10s | %-19s |\n", "ID", "NAMA", "GENDER", "GAJI");
printf("+--------+---------------------------+------------+---------------------+\n");
```
Menampilkan header tabel untuk data pegawai dengan format yang rapi.

```
double total_gaji = 0;
int jumlah_data = 0;
char formatted_gaji[20];
```
Mendeklarasikan variabel untuk menghitung total gaji (total_gaji), menghitung jumlah data (jumlah_data), dan menampung format gaji yang sudah diformat (formatted_gaji).Mendeklarasikan variabel untuk menghitung total gaji (total_gaji), menghitung jumlah data (jumlah_data), dan menampung format gaji yang sudah diformat (formatted_gaji).

```
if (pakai_indeks) {
    for (int i = 0; i < jumlah_indeks; i++) {
        fseek(file_data, daftar_indeks[i].posisi, SEEK_SET);
        Pegawai pegawai;
        fread(&pegawai, sizeof(Pegawai), 1, file_data);
        
        if (pegawai.is_deleted) continue;
        
        format_gaji(pegawai.gaji, formatted_gaji);
        printf("| %-6s | %-25s | %-10s | Rp. %15s |\n", 
               pegawai.id, 
               pegawai.nama, 
               pegawai.gender ? "Laki" : "Perempuan", 
               formatted_gaji);
        total_gaji += pegawai.gaji;
        jumlah_data++;
        
        if (jumlah_data % 20 == 0) {
            printf("+--------+---------------------------+------------+---------------------+\n");
            printf("Tekan enter untuk melanjutkan...");
            getch();
            printf("\n");
            printf("| %-6s | %-25s | %-10s | %-19s |\n", "ID", "NAMA", "GENDER", "GAJI");
            printf("+--------+---------------------------+------------+---------------------+\n");
        }
    }
}
```
Jika pakai_indeks bernilai true, fungsi ini menampilkan data pegawai dengan menggunakan indeks.
fseek(): Menentukan posisi data pegawai berdasarkan posisi yang ada dalam indeks.
fread(): Membaca data pegawai dari file.
pegawai.is_deleted: Memeriksa apakah pegawai sudah dihapus, jika ya, data tersebut dilewatkan.
format_gaji(): Memformat gaji pegawai untuk ditampilkan dengan format yang sesuai.
printf(): Menampilkan data pegawai.
total_gaji: Menambahkan gaji pegawai ke total gaji.
jumlah_data: Menghitung jumlah data pegawai yang ditampilkan.
if (jumlah_data % 20 == 0): Setiap 20 data yang ditampilkan, program meminta pengguna menekan enter untuk melanjutkan.

```
else {
    rewind(file_data);
    Pegawai pegawai;
    while (fread(&pegawai, sizeof(Pegawai), 1, file_data)) {
        if (pegawai.is_deleted) continue;
        
        format_gaji(pegawai.gaji, formatted_gaji);
        printf("| %-6s | %-25s | %-10s | Rp. %15s |\n", 
               pegawai.id, 
               pegawai.nama, 
               pegawai.gender ? "Laki" : "Perempuan", 
               formatted_gaji);
        total_gaji += pegawai.gaji;
        jumlah_data++;
        
        if (jumlah_data % 20 == 0) {
            printf("+--------+---------------------------+------------+---------------------+\n");
            printf("Tekan enter untuk melanjutkan...");
            getch();
            printf("\n");
            printf("| %-6s | %-25s | %-10s | %-15s |\n", "ID", "NAMA", "GENDER", "GAJI");
            printf("+--------+---------------------------+------------+---------------------+\n");
        }
    }
}
```
Jika pakai_indeks bernilai false, fungsi ini akan menampilkan data tanpa menggunakan indeks.
rewind(file_data): Mengatur ulang pointer file ke awal.
fread(): Membaca data pegawai dari file secara langsung.
pegawai.is_deleted: Memeriksa apakah pegawai sudah dihapus, jika ya, data tersebut dilewatkan.
printf(): Menampilkan data pegawai dengan format yang rapi.
total_gaji dan jumlah_data: Menambahkan gaji pegawai ke total gaji dan menghitung jumlah data.
if (jumlah_data % 20 == 0): Setiap 20 data yang ditampilkan, program meminta pengguna menekan enter untuk melanjutkan.

```
char formatted_total[20];
format_gaji(total_gaji, formatted_total);
printf("+--------+---------------------------+------------+---------------------+\n");
printf("| %43s     | Rp. %15s |\n", "TOTAL GAJI PEGAWAI", formatted_total);
printf("+--------+---------------------------+------------+---------------------+\n");
```
Menghitung dan menampilkan total gaji pegawai yang sudah ditampilkan.
format_gaji(): Memformat total gaji untuk ditampilkan.
printf(): Menampilkan total gaji dengan format yang rapi.

## FUNGSI UTAMA

```
file_data = fopen("pegawai.dat", "rb+");
if (!file_data) file_data = fopen("pegawai.dat", "wb+");
if (!file_data) {
    printf("Gagal membuka file data!");
    return 1;
}
```
fopen("pegawai.dat", "rb+"): Membuka file pegawai.dat untuk dibaca dan ditulis (binary mode). Jika file tidak ada, maka dilanjutkan ke baris berikutnya.
fopen("pegawai.dat", "wb+"): Jika file tidak ditemukan, membuka file baru dengan mode baca dan tulis (binary mode).
if (!file_data): Jika file gagal dibuka, menampilkan pesan kesalahan dan keluar dengan status 1.

```
muat_indeks();
```
Memanggil fungsi muat_indeks() untuk memuat data indeks pegawai dari file indeks.dat.

```
char pilihan;
do {
```
Mendeklarasikan variabel pilihan untuk menyimpan input menu yang dipilih oleh pengguna. Dimulai dengan do-while loop untuk menjalankan menu hingga pengguna memilih keluar.

```
system("cls");
printf("SISTEM PENGELOLAAN DATA PEGAWAI\n");
printf("===============================\n");
printf("A. Tambah Data\n");
printf("U. Ubah Data\n");
printf("D. Tampilkan Data (dengan Indeks)\n");
printf("T. Tampilkan Data (tanpa Indeks)\n");
printf("H. Hapus Data\n");
printf("K. Keluar\n");
printf("Pilihan [A/U/D/T/H/K]: ");
```
system("cls"): Membersihkan layar terminal agar tampilan menu lebih bersih.
Menampilkan menu pilihan dengan opsi untuk tambah, ubah, tampilkan, hapus data, atau keluar.

```
scanf(" %c", &pilihan);
bersihkan_buffer();
```
scanf(" %c", &pilihan): Membaca karakter input dari pengguna untuk memilih menu.
bersihkan_buffer(): Fungsi untuk membersihkan buffer input agar tidak ada karakter sisa yang terbaca.

```
pilihan = toupper(pilihan);
```
toupper(pilihan): Mengubah input pengguna menjadi huruf kapital agar perbandingan lebih mudah.

```
switch (pilihan) {
    case 'A': tambah_data(); break;
    case 'U': ubah_data(); break;
    case 'D': tampil_data(1); break;
    case 'T': tampil_data(0); break;
    case 'H': hapus_data(); break;
    case 'K': break;
    default: printf("Pilihan tidak valid!\n");
}
```
switch (pilihan): Memeriksa nilai pilihan dan mengeksekusi fungsi sesuai dengan opsi yang dipilih:
A: Menambah data pegawai.
U: Mengubah data pegawai.
D: Menampilkan data pegawai dengan indeks.
T: Menampilkan data pegawai tanpa indeks.
H: Menghapus data pegawai.
K: Keluar dari menu.
default: Jika input tidak valid, menampilkan pesan error.

```
if (pilihan != 'K') {
    printf("\nTekan enter untuk melanjutkan...");
    getch();
}
```
if (pilihan != 'K'): Jika pengguna tidak memilih keluar, program meminta pengguna menekan enter untuk melanjutkan.

```
fclose(file_data);
```
Menutup file pegawai.dat setelah semua operasi selesai.

```
return 0;
```
Program selesai, mengembalikan nilai 0 sebagai tanda bahwa program berjalan dengan sukses.


