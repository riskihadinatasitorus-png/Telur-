import hashlib

class TelurWalletInterface:
    def __init__(self):
        self.session_key = None
        self.session_address = None
        self.address_book = {}

    def start_terminal(self, engine):
        while True:
            print("\n" + "="*40)
            print("🍳  TELUR CLIENT TERMINAL v3.0  🍳")
            print("="*40)
            if self.session_address:
                fiat = engine.fiat_balances.get(self.session_address, 0.0)
                coin = engine.vault_balances.get(self.session_address, 0.0)
                print(f"ALAMAT : {self.session_address}")
                print(f"SALDO  : Rp {fiat:,.0f} | {coin:,} TLR")
            else:
                print("STATUS : DISCONNECTED")
            print("-"*40)
            print("1. Login Dompet (Input Kunci Privat)")
            print("2. Kirim Koin ke Alamat Lain")
            print("3. Beli Koin dari Toko")
            print("4. Jual Koin ke Toko")
            print("5. Simpan Alamat Baru")
            print("6. Lihat Buku Alamat")
            print("7. Keluar")
            print("="*40)
            
            cmd = input("Pilih menu (1-7): ").strip()

            if cmd == "1":
                key = input("Masukkan Kunci Privat: ").strip()
                if key.startswith("priv_"):
                    self.session_key = key
                    self.session_address = "tlr_" + hashlib.sha256(key.encode()).hexdigest()[:20]
                    # Pastikan akun terdaftar di data fiat/koin jika belum ada
                    if self.session_address not in engine.fiat_balances:
                        engine.fiat_balances[self.session_address] = 20000000.0
                        engine.vault_balances[self.session_address] = 0.0
                    print("[OK] Login berhasil.")
                else:
                    print("[ERR] Format kunci salah.")
            elif cmd == "2":
                if not self.session_key: print("[ERR] Login dulu!"); continue
                to_addr = input("Alamat tujuan: ").strip()
                amt = float(input("Jumlah TLR: "))
                _, msg = engine.execute_ledger_transfer(self.session_key, to_addr, amt)
                print(f"[LOG] {msg}")
            elif cmd == "3":
                if not self.session_key: print("[ERR] Login dulu!"); continue
                fiat_amt = float(input("Jumlah Rupiah untuk beli: Rp "))
                _, msg = engine.execute_shop_buy(self.session_key, fiat_amt)
                print(f"[LOG] {msg}")
            elif cmd == "4":
                if not self.session_key: print("[ERR] Login dulu!"); continue
                coin_amt = float(input("Jumlah TLR untuk dijual: "))
                _, msg = engine.execute_shop_sell(self.session_key, coin_amt)
                print(f"[LOG] {msg}")
            elif cmd == "5":
                name = input("Nama pemilik: ").strip()
                addr = input("Alamat koin (tlr_...): ").strip()
                if addr.startswith("tlr_"):
                    self.address_book[name] = addr
                    print("[OK] Alamat disimpan.")
                else:
                    print("[ERR] Alamat tidak valid.")
            elif cmd == "6":
                print("\n📒 BUKU ALAMAT:")
                for name, addr in self.address_book.items():
                    print(f"👤 {name}: {addr}")
            elif cmd == "7":
                print("Terminal ditutup."); break

if __name__ == "__main__":
    from coin_telur import TelurCoreEngine
    core = TelurCoreEngine()
    client = TelurWalletInterface()
    client.start_terminal(core)
    




