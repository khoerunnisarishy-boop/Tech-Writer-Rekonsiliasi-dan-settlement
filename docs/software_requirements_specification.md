%% Subprocess: 1. Pengumpulan Data
    subgraph Data_Ingestion["1. Pengumpulan Data"]
        Start1(("Mulai Ingestion"))
        T1_1["Unggah File Mutasi & Log Transaksi"]
        T1_2["Parsing & Normalisasi Format Data"]
        End1(("Data Siap"))

        Start1 --> T1_1
        T1_1 --> T1_2
        T1_2 --> End1
    end

    %% Subprocess: 2. Pemadanan Data
    subgraph Data_Matching["2. Pemadanan Data (Reconciliation)"]
        Start2(("Mulai Matching"))
        T2_1["Bandingkan Log Internal vs Bank Statement"]
        G2_1{"Data Cocok?"}
        T2_2["Tandai Transaksi Matched"]
        T2_3["Tandai Discrepancy / Unmatched"]
        T2_4["Review & Investigasi Manual"]
        End2(("Rekonsiliasi Selesai"))

        Start2 --> T2_1
        T2_1 --> G2_1
        G2_1 -- Ya --> T2_2
        G2_1 -- Tidak --> T2_3
        T2_3 --> T2_4
        T2_4 --> T2_2
        T2_2 --> End2
    end

    %% Subprocess: 3. Penyelesaian Keuangan
    subgraph Settlement["3. Penyelesaian Keuangan (Settlement)"]
        Start3(("Mulai Settlement"))
        T3_1["Kalkulasi Total Netting & Potongan Fee"]
        T3_2["Generate Instruksi Payout / Bulk Transfer"]
        G3_1{"Eksekusi Sukses?"}
        T3_3["Update Status Settlement: Sukses"]
        T3_4["Penanganan Transaksi Gagal / Pending"]
        End3(("Settlement Selesai"))

        Start3 --> T3_1
        T3_1 --> T3_2
        T3_2 --> G3_1
        G3_1 -- Ya --> T3_3
        G3_1 -- Tidak --> T3_4
        T3_4 --> T3_2
        T3_3 --> End3
    end

    %% Subprocess: 4. Jurnal & Pelaporan
    subgraph Journaling_Reporting["4. Jurnal & Pelaporan"]
        Start4(("Mulai Reporting"))
        T4_1["Generate Posting Jurnal Otomatis ke GL/ERP"]
        T4_2["Buat Laporan Rekonsiliasi & Audit Trail"]
        End4(("Proses Selesai"))

        Start4 --> T4_1
        T4_1 --> T4_2
        T4_2 --> End4
    end

    %% Alur Antar Tahapan
    End1 --> Start2
    End2 --> Start3
    End3 --> Start4