import hashlib
import time
import secrets

class CoinTelurAdvanced:
    def __init__(self):
        self.blockchain = []
        self.dompet_jaringan = {}
        self.total_suplai = 50000000.0
        self.suplai_beredar = 0.0
        self.gas_fee_standar = 5.0  # Biaya transfer flat: 5 Coin Telur per transaksi
        
        # INI AKUN UTAMA ANDA
        self.kunci_privat_anda = "priv_" + hashlib.sha256(b"nmara5220@gmail.com-190619").hexdigest()[:32]
        self.alamat_publik_anda = "tlr_" + hashlib.sha256(self.kunci_privat_anda.encode()).hexdigest()[:20]
        
        # Alokasi saldo awal
        self.dompet_jaringan[self.alamat_publik_anda] = 15000000.0
        self.suplai_beredar += 15000000.0
        
        # Buat Genesis Block
        self.kunci_blok_baru(previous_hash="0", data="Genesis Block - Sistem Fitur Lanjutan Aktif")

    def buat_dompet_baru(self):
        kunci_privat_baru = "priv_" + secrets.token_hex(16)
        alamat_publik_baru = "tlr_" + hashlib.sha256(kunci_privat_baru.encode()).hexdigest()[:20]
        self.dompet_jaringan[alamat_publik_baru] = 0.0
        return kunci_privat_baru, alamat_publik_baru

    def kirim_coin_dengan_gas(self, kunci_privat_pengirim, alamat_tujuan, jumlah, alamat_penambang):
        """Memproses pengiriman koin dengan potongan Gas Fee untuk Penambang"""
        alamat_pengirim = "tlr_" + hashlib.sha256(kunci_privat_pengirim.encode()).hexdigest()[:20]
        
        if alamat_pengirim not in self.dompet_jaringan:
            return False, "❌ Kunci Privat Tidak Valid!"
            
        # Total yang harus dibayar pengirim = Jumlah kirim + Gas Fee
        total_potongan = jumlah + self.gas_fee_standar
        
        if self.dompet_jaringan[alamat_pengirim] < total_potongan:
            return False, f"❌ Saldo kurang! Butuh {total_potongan:,} (Termasuk Gas Fee: {self.gas_fee_standar} TLR)"
            
        if alamat_tujuan not in self.dompet_jaringan:
            return False, "❌ Alamat tujuan tidak ditemukan!"

        # Eksekusi Ledger (Potong pengirim, tambah penerima, beri upah gas ke penambang)
        self.dompet_jaringan[alamat_pengirim] -= total_potongan
        self.dompet_jaringan[alamat_tujuan] += jumlah
        self.dompet_jaringan[alamat_penambang] = self.dompet_jaringan.get(alamat_penambang, 0.0) + self.gas_fee_standar
        
        # Kunci transaksi ke blok baru
        detail_tx = f"TX: {alamat_pengirim[:8]}.. kirim {jumlah:,} ke {alamat_tujuan[:8]}.. | Gas: {self.gas_fee_standar} ke {alamat_penambang[:8]}.."
        self.kunci_blok_baru(previous_hash=self.blockchain[-1]["hash"], data=detail_tx)
        return True, "💸 Transaksi Sukses Masuk Blok!"

    def cek_riwayat_alamat(self, alamat_dompet):
        """FITUR BARU: Melacak semua riwayat blok yang melibatkan alamat ini"""
        print(f"\n🔍 MENELUSURI BLOK UNTUK ALAMAT: {alamat_dompet}")
        ditemukan = False
        
        for blok in self.blockchain:
            # Jika alamat dompet tersebut tertulis di dalam data blok
            if alamat_dompet[:8] in blok["data"] or alamat_dompet in blok["data"]:
                print(f"📌 Terdeteksi di Blok #{blok['index']} [{blok['timestamp']}]")
                print(f"   Aktivitas: {blok['data']}")
                ditemukan = True
        
        if not ditemukan:
            print("❌ Belum ada riwayat transaksi untuk alamat ini.")

    def kunci_blok_baru(self, previous_hash, data):
        index = len(self.blockchain)
        timestamp = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
        teks_kunci = f"{index}{timestamp}{data}{previous_hash}"
        hash_blok = hashlib.sha256(teks_kunci.encode()).hexdigest()
        
        self.blockchain.append({
            "index": index,
            "timestamp": timestamp,
            "data": data,
            "previous_hash": previous_hash,
            "hash": hash_blok
        })

# =======================================================
# SIMULASI PENGUJIAN FITUR BARU DI BLOK SENDIRI
# =======================================================
coin_telur = CoinTelurAdvanced()

# 1. Buat dua akun baru (Satu pengguna, satu bertindak sebagai penambang)
_, dompet_budi = coin_telur.buat_dompet_baru()
_, dompet_penambang = coin_telur.buat_dompet_baru()

print(f"Saldo Awal Anda       : {coin_telur.dompet_jaringan[coin_telur.alamat_publik_anda]:,} TLR")
print(f"Saldo Awal Penambang  : {coin_telur.dompet_jaringan[dompet_penambang]:,} TLR")

# 2. Anda mengirim 100.000 koin ke Budi, biaya gas mengalir ke akun Penambang
print("\n⚡ Memproses transfer pertama dengan skema Gas Fee...")
coin_telur.kirim_coin_dengan_gas(coin_telur.kunci_privat_anda, dompet_budi, 100000, dompet_penambang)

print(f" Sisa Saldo Anda      : {coin_telur.dompet_jaringan[coin_telur.alamat_publik_anda]:,} TLR")
print(f" Saldo Upah Penambang : {coin_telur.dompet_jaringan[dompet_penambang]:,} TLR (Berhasil menerima 5 TLR!)")

# 3. MENGUJI FITUR EXPLORER (RIWAYAT TRANSAKSI)
coin_telur.cek_riwayat_alamat(coin_telur.alamat_publik_anda)
