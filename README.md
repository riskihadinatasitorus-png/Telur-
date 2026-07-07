# =======================================================
# MENU DOMPET INTERAKTIF UNTUK MENGECEK KOIN ANDA
# =======================================================

print("\n=== 💳 DOMPET DIGITAL COIN TELUR ===")
input_email = input("Masukkan Email Akun Anda: ")
input_sandi = input("Masukkan Sandi Akun Anda: ")

# Proses pengecekan keamanan sebelum membuka dompet
if coin_telur.masuk_akun(input_email, input_sandi)[0]:
    saldo_simpanan = coin_telur.akun[input_email]["saldo"]
    alamat_dompet = coin_telur.akun[input_email]["alamat"]
    print(f"\n🔐 AKSES DOMPET DITERIMA!")
    print(f"📍 Alamat Dompet Anda : {alamat_dompet}")
    print(f"💰 Total Koin Tersimpan: {saldo_simpanan:,} Coin Telur")
else:
    print("\n❌ AKSES DITOLAK! Email atau sandi salah.")
    ## 🪙 Proyek Coin Telur
Model blockchain ini memiliki spesifikasi khusus:
- Total suplai dibatasi maksimal 50 Juta Coin.
- Dilengkapi sistem akun penambang, voucher, dan penanda log in/out.
- Setiap transaksi otomatis dikunci ke dalam blok menggunakan enkripsi SHA-256.
- 
import hashlib
import time

class CoinTelurBlockchain:
    def __init__(self):
        self.blockchain = []
        self.akun = {}
        self.voucher = {}
        self.total_suplai = 50000000.0  # Batas Maksimal: 50 Juta Coin
        self.suplai_beredar = 0.0
        
        # 1. SETTING AKUN PERTAMA OTOMATIS
        self.buat_daftar_akun("nmara5220@gmail.com", "190619")
        self.akun["nmara5220@gmail.com"]["saldo"] = 15000000.0  # Alokasi awal 15 Juta Coin
        self.suplai_beredar += 15000000.0
        
        # Buat Genesis Block (Kunci Blok Awal)
        self.kunci_blok_baru(previous_hash="0", pencatat="SISTEM", data="Genesis Block - Inisialisasi Coin Telur")

    def buat_daftar_akun(self, email, sandi):
        if email in self.akun:
            return False, "⚠️ Email sudah terdaftar!"
        
        # Alamat koin dibuat unik terenkripsi otomatis berdasarkan email
        alamat_koin = "tlr_" + hashlib.md5(email.encode()).hexdigest()[:10]
        self.akun[email] = {
            "sandi": sandi,
            "alamat": alamat_koin,
            "saldo": 0.0,
            "status_login": False
        }
        return True, f"✅ Akun berhasil didaftarkan! Alamat Coin Anda: {alamat_koin}"

    def masuk_akun(self, email, sandi):
        if email not in self.akun:
            return False, "⚠️ Email tidak ditemukan!"
        if self.akun[email]["sandi"] != sandi:
            return False, "⚠️ Kata sandi salah!"
        
        self.akun[email]["status_login"] = True
        return True, f"🔓 Berhasil masuk ke akun: {email}"

    def keluar_akun(self, email):
        if email in self.akun and self.akun[email]["status_login"]:
            self.akun[email]["status_login"] = False
            return True, f"🔒 Berhasil keluar dari akun: {email}"
        return False, "⚠️ Gagal keluar. Akun belum login atau tidak ditemukan."

    def kirim_terima_coin(self, email_pengirim, alamat_penerima, jumlah):
        # Proteksi keamanan: wajib masuk akun
        if not self.akun.get(email_pengirim, {}).get("status_login", False):
            return False, "❌ Gagal! Anda harus MASUK AKUN terlebih dahulu."
        
        if self.akun[email_pengirim]["saldo"] < jumlah:
            return False, "❌ Gagal! Saldo tidak mencukupi untuk mengirim."
        
        # Cari data email pemilik alamat penerima
        email_penerima = None
        for em, data in self.akun.items():
            if data["alamat"] == alamat_penerima:
                email_penerima = em
                break
                
        if not email_penerima:
            return False, "❌ Gagal! Alamat koin penerima tidak valid."
            
        # Proses Transfer Saldo (Kirim & Terima)
        self.akun[email_pengirim]["saldo"] -= jumlah
        self.akun[email_penerima]["saldo"] += jumlah
        
        # Kunci transaksi ke dalam blok baru
        data_transaksi = f"Transfer: {email_pengirim} mengirim {jumlah:,} Coin Telur ke {alamat_penerima}"
        self.kunci_blok_baru(previous_hash=self.blockchain[-1]["hash"], pencatat=email_pengirim, data=data_transaksi)
        return True, f"💸 Sukses mengirim {jumlah:,} coin ke {alamat_penerima}!"

    def buat_mode_voucher(self, email_pembuat, jumlah):
        if not self.akun.get(email_pembuat, {}).get("status_login", False):
            return False, "❌ Gagal! Harus masuk akun terlebih dahulu."
            
        # Validasi pembatasan total suplai global agar tidak lewat 50 juta
        if self.suplai_beredar + jumlah > self.total_suplai:
            return False, "❌ Gagal! Pencetakan voucher melebihi batas total suplai 50 juta Coin Telur."
            
        kode_voucher = "VCHR-TLR-" + hashlib.md5(f"{time.time()}{jumlah}".encode()).hexdigest()[:6].upper()
        self.voucher[kode_voucher] = {
            "jumlah": jumlah,
            "status": "AKTIF"
        }
        self.suplai_beredar += jumlah
        return True, f"🎟️ Voucher Sukses Dibuat! Kode: {kode_voucher} senilai {jumlah:,} Coin Telur."

    def kunci_blok_baru(self, previous_hash, pencatat, data):
        index = len(self.blockchain)
        timestamp = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
        
        # Mengunci blok menggunakan fungsi enkripsi SHA-256
        teks_kunci = f"{index}{timestamp}{data}{previous_hash}{pencatat}"
        hash_blok = hashlib.sha256(teks_kunci.encode()).hexdigest()
        
        blok = {
            "index": index,
            "timestamp": timestamp,
            "data": data,
            "pencatat_blok": pencatat,
            "previous_hash": previous_hash,
            "hash": hash_blok
        }
        self.blockchain.append(blok)
        return blok

# =======================================================
# SIMULASI JALANNYA SISTEM COIN TELUR SECARA OTOMATIS
# =======================================================

coin_telur = CoinTelurBlockchain()

print("=== 1. STRUKTUR AKUN PERTAMA (ALOKASI AWAL) ===")
print(f"Total Limit Suplai Jaringan : {coin_telur.total_suplai:,} Coin")
print(f"Email Akun                  : nmara5220@gmail.com")
print(f"Sandi Akun                  : {coin_telur.akun['nmara5220@gmail.com']['sandi']}")
print(f"Alamat Dompet               : {coin_telur.akun['nmara5220@gmail.com']['alamat']}")
print(f"Saldo Awal                  : {coin_telur.akun['nmara5220@gmail.com']['saldo']:,} Coin")

print("\n=== 2. MENCOBA LOGIN (MASUK AKUN) ===")
_, hasil_login = coin_telur.masuk_akun("nmara5220@gmail.com", "190619")
print(hasil_login)

print("\n=== 3. MEMBUAT AKUN PENAMBANG BARU ===")
_, hasil_daftar = coin_telur.buat_daftar_akun("penambang_baru@gmail.com", "sandi123")
print(hasil_daftar)
alamat_penambang_baru = coin_telur.akun["penambang_baru@gmail.com"]["alamat"]

print("\n=== 4. PROSES KIRIM & TERIMA COIN ===")
# Akun pertama mengirim 5 Juta Coin ke akun penambang baru
_, hasil_kirim = coin_telur.kirim_coin_terima("nmara5220@gmail.com", alamat_penambang_baru, 5000000)
print(hasil_kirim)
print(f"-> Sisa Saldo Akun Pertama : {coin_telur.akun['nmara5220@gmail.com']['saldo']:,} Coin")
print(f"-> Saldo Penambang Baru    : {coin_telur.akun['penambang_baru@gmail.com']['saldo']:,} Coin")

print("\n=== 5. MODE VOUCHER (MENCETAK COIN BARU DALAM BENTUK VOUCHER) ===")
# Akun pertama membuat voucher bernilai 15 Juta Coin
_, hasil_voucher = coin_telur.buat_mode_voucher("nmara5220@gmail.com", 15000000)
print(hasil_voucher)
print(f"Total Koin Beredar Sekarang: {coin_telur.suplai_beredar:,} / {coin_telur.total_suplai:,} Coin")

print("\n=== 6. DAFTAR BLOK YANG BERHASIL DIKUNCI (BLOCKCHAIN) ===")
for blok in coin_telur.blockchain:
    print(f"📦 BLOK #{blok['index']} | Waktu: {blok['timestamp']}")
    print(f"   Aktivitas : {blok['data']}")
    print(f"   Pencatat  : {blok['pencatat_blok']}")
    print(f"   Hash Kunci: {blok['hash']}")
    print("-" * 65)
