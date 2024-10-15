# RTOSIntro

RTOS Introduction with pickButtonTask, getADCTask, dispUARTTask, and ctrlLEDTask

![RTOS Task Diagram](Task_Diagram.jpg)

This diagram illustrates the relationships between different tasks in an RTOS (Real-Time Operating System) setup:

- **PickButtonTask**: Handles button input
- **getADCTask**: Manages Analog-to-Digital Conversion
- **ctrlLEDTask**: Controls LED output
- **dispUARTTask**: Manages UART display output

Dalam sistem berbasis *Real-Time Operating System* (RTOS), setiap *task* atau *thread* berjalan secara *concurrent* (bersamaan) tetapi dikelola secara cermat oleh RTOS scheduler. Salah satu elemen penting dalam pengelolaan *task* oleh RTOS adalah **prioritas task**. Prioritas ini menentukan urutan di mana tugas dijalankan ketika beberapa tugas siap untuk dijalankan pada saat yang sama.

###Prioritas dan Hubungan Antar Task di Kode
Dalam kode yang kamu berikan, terdapat lima *task* yang dibuat dengan prioritas yang berbeda-beda, seperti berikut:

1. **defaultTask** - Prioritas Normal
2. **pickButtonTask** - Prioritas Rendah
3. **getADCTask** - Prioritas Di Atas Normal (*AboveNormal*)
4. **dispUARTTask** - Prioritas Normal
5. **ctrlLEDTask** - Prioritas Normal

**Prioritas** menentukan kapan *task* ini akan dijalankan jika lebih dari satu *task* siap dijalankan.

### Prinsip Prioritas RTOS

- **Tugas dengan prioritas lebih tinggi selalu berjalan lebih dulu**. Jika tugas dengan prioritas tinggi siap, RTOS akan menghentikan tugas dengan prioritas lebih rendah dan menjalankan tugas dengan prioritas yang lebih tinggi.
- **Tugas dengan prioritas yang sama akan dijalankan secara bergantian** sesuai dengan mekanisme *time-slicing* atau berdasarkan apakah mereka diblokir atau menunggu suatu kejadian.

### Hubungan Antara Task-Task di Kode

1. **defaultTask**: 
   - Prioritasnya adalah *normal* (standar). Tugas ini tidak memiliki fungsi khusus selain mungkin sebagai *placeholder* atau tugas utama yang berjalan terus-menerus dengan sedikit operasi.

2. **pickButtonTask** (Prioritas Rendah):
   - Tugas ini bertugas untuk memantau status tombol fisik (Button1 dan Button2).
   - Karena prioritasnya rendah, tugas ini akan dijalankan hanya ketika tidak ada tugas dengan prioritas lebih tinggi yang siap dijalankan. Tugas ini mengandalkan delay (*debounce*) untuk memastikan bahwa pembacaan tombol dilakukan dengan benar.

3. **getADCTask** (Prioritas Di Atas Normal):
   - Tugas ini membaca nilai dari ADC (Analog-to-Digital Converter). Karena tugas ini memiliki prioritas yang lebih tinggi (*Above Normal*), tugas ini akan berjalan lebih sering dan akan diutamakan oleh scheduler dibandingkan tugas lain dengan prioritas normal atau rendah.
   - Fungsi ini penting karena sensor yang terhubung ke ADC mungkin perlu diperbarui secara cepat untuk menjaga keakuratan data.

4. **dispUARTTask** (Prioritas Normal):
   - Tugas ini mengirimkan data ADC dan status tombol melalui UART (komunikasi serial). Karena prioritasnya *normal*, ia akan berjalan ketika tidak ada tugas dengan prioritas lebih tinggi (seperti `getADCTask`).

5. **ctrlLEDTask** (Prioritas Normal):
   - Tugas ini mengontrol LED berdasarkan status tombol dan nilai ADC. Jika tombol ditekan atau nilai ADC melebihi batas tertentu, LED akan di-toggle atau dimatikan.
   - Prioritasnya juga *normal*, sehingga akan berjalan secara bergantian dengan `dispUARTTask` dan `defaultTask`.

### Hubungan Prioritas
- Karena `getADCTask` memiliki prioritas tertinggi, tugas ini akan dijalankan segera ketika siap dan dapat menginterupsi tugas dengan prioritas lebih rendah seperti `pickButtonTask` atau `dispUARTTask`.
- `pickButtonTask` memiliki prioritas terendah, artinya tugas ini hanya akan berjalan ketika semua tugas lain (baik `getADCTask`, `dispUARTTask`, `ctrlLEDTask`, atau `defaultTask`) tidak siap atau sedang menunggu (*blocked*).
- `dispUARTTask` dan `ctrlLEDTask` memiliki prioritas yang sama (normal), sehingga mereka akan bergantian dijalankan oleh scheduler berdasarkan mekanisme *time-slicing* atau keadaan masing-masing tugas.

### Skema Eksekusi RTOS
1. RTOS akan melihat tugas dengan prioritas tertinggi yang siap untuk dijalankan. Jika ada tugas dengan prioritas lebih tinggi yang siap, tugas yang sedang berjalan akan dihentikan sementara (pre-empted).
2. Jika beberapa tugas memiliki prioritas yang sama, RTOS akan membagi waktu secara merata atau menjalankan tugas yang sudah paling lama menunggu (*round-robin*).
3. Jika tidak ada tugas lain yang siap, tugas dengan prioritas rendah seperti `pickButtonTask` akan dijalankan.

Dalam hal ini, RTOS memastikan bahwa tugas penting seperti pembacaan ADC (`getADCTask`) selalu diproses lebih dulu, sementara tugas pemantauan tombol atau *default task* diproses ketika sistem memiliki kapasitas yang lebih rendah.

Ini adalah cara RTOS memberikan respons real-time yang cepat untuk tugas-tugas prioritas tinggi tanpa menghentikan tugas prioritas lebih rendah yang kurang penting.
