import hashlib
import secrets

class TelurWalletInterface:
    def __init__(self):
        self.session_key = None
        self.session_address = None
        # FITUR BARU: Buku alamat untuk menyimpan alamat koin
        self.address_book = {} 

    def start_terminal(self, engine_instance):
        while True:
            print("\n" + "="*45)
            print("🚀  TELUR NETWORK TERMINAL CLIENT v2.0  🚀")
            print("="*45)
            if self.session_address:
                print(f"STATUS: ACTIVE SESSION")
                print(f"NODE ADDRESS: {self.session_address}")
                # Menampilkan saldo Rupiah dan saldo Koin sekaligus
                fiat = engine_instance.fiat_balances.get(self.session_address, 0.0)
                coin = engine_instance.vault_balances.get(self.session_address, 0.0)
                print(f"BALANCE     : Rp {fiat:,.0f} | {coin:,} TLR")
            else:
                print("STATUS: DISCONNECTED")
            print("-"*45)
            print("1. Initialize New Wallet Keypair (Buat Dompet)")
            print("2. Authenticate with Existing Secret Key (Masuk)")
            print("3. Broadcast Transaction (Kirim Koin ke Orang)")
            print("4. Buy Coins from Shop (Beli Koin dari Toko)")
            print("5. Sell Coins to Shop (Jual Koin ke Toko)")
            print("6. Save Address to Book (Simpan Alamat Koin)")
            print("7. View Saved Address Book (Lihat Daftar Alamat)")
            print("8. Terminate Terminal (Keluar)")
            print("="*45)
            
            cmd = input("Enter command (1-8): ").strip()

            if cmd == "1":
                self.init_keypair(engine_instance)
            elif cmd == "2":
                self.auth_session()
            elif cmd == "3":
                self.broadcast_tx(engine_instance)
            elif cmd == "4":
                self.buy_from_shop(engine_instance)
            elif cmd == "5":
                self.sell_to_shop(engine_instance)
            elif cmd == "6":
                self.save_address()
            elif cmd == "7":
                self.view_address_book()
            elif cmd == "8":
                print("\n[INFO] Terminal session terminated.")
                break
            else:
                print("[WARN] Invalid command sequence.")

    def init_keypair(self, engine):
        secret_seed = "priv_" + secrets.token_hex(16)
        public_node = "tlr_" + hashlib.sha256(secret_seed.encode()).hexdigest()[:20]
        engine.vault_balances[public_node] = 0.0
        engine.fiat_balances[public_node] = 20000000.0  # Modal awal Rp 20 Juta
        self.session_key = secret_seed
        self.session_address = public_node
        print("\n[SUCCESS] NEW KEYPAIR GENERATED")
        print(f"Public Address : {public_node}")
        print(f"Secret Key     : {secret_seed}")

    def auth_session(self):
        token_input = input("\nEnter Secret Key (priv_...): ").strip()
        if not token_input.startswith("priv_") or len(token_input) < 20:
            print("[ERROR] Authentication token malformed.")
            return
        self.session_address = "tlr_" + hashlib.sha256(token_input.encode()).hexdigest()[:20]
        self.session_key = token_input
        print(f"\n[SUCCESS] Authenticated. Active node: {self.session_address}")

    def broadcast_tx(self, engine):
        if not self.session_key:
            print("[ERROR] Authenticate session first.")
            return
        target = input("\nEnter Destination Address (tlr_...): ").strip()
        try:
            amount = float(input("Enter Value to Transfer: "))
        except ValueError:
            print("[ERROR] Value must be numerical.")
            return
        miner_node = "tlr_network_central_pool"
        status, log_msg = engine.execute_ledger_transfer(self.session_key, target, amount, miner_node)
        print(f"[NODE LOG] {log_msg}")

    def buy_from_shop(self, engine):
        """MENU ANGKA 4: Membeli koin dari toko otomatis"""
        if not self.session_key:
            print("[ERROR] Authenticate session first.")
            return
        try:
            fiat_spent = float(input(f"\nHarga 1 TLR = Rp {engine.token_price_idr:,.0f}\nMasukkan jumlah Rupiah untuk beli koin: Rp "))
        except ValueError:
            print("[ERROR] Input must be numerical.")
            return
        status, log_msg = engine.execute_shop_buy(self.session_key, fiat_spent)
        print(f"[NODE LOG] {log_msg}")

    def sell_to_shop(self, engine):
        """MENU ANGKA 5: Menjual koin ke toko otomatis"""
        if not self.session_key:
            print("[ERROR] Authenticate session first.")
            return
        try:
            token_sold = float(input("\nMasukkan jumlah koin TLR yang ingin dijual: "))
        except ValueError:
            print("[ERROR] Input must be numerical.")
            return
        status, log_msg = engine.execute_shop_sell(self.session_key, token_sold)
        print(f"[NODE LOG] {log_msg}")

    def save_address(self):
        """MENU ANGKA 6: Menyimpan alamat koin ke buku alamat"""
        name = input("\nMasukkan nama pemilik alamat (Contoh: Budi): ").strip()
        address = input("Masukkan alamat koin (tlr_...): ").strip()
        if not address.startswith("tlr_"):
            print("[ERROR] Alamat harus berawalan 'tlr_'")
            return
        self.address_book[name] = address
        print(f"[SUCCESS] Alamat koin {name} berhasil disimpan!")

    def view_address_book(self):
        """MENU ANGKA 7: Melihat daftar alamat yang disimpan"""
        print("\n📒 BUKU ALAMAT COIN TELUR Anda:")
        if not self.address_book:
            print(" (Buku alamat masih kosong) ")
            return
        for name, address in self.address_book.items():
            print(f"👤 {name} : {address}")

if __name__ == "__main__":
    from coin_telur import TelurCoreEngine
    core_network = TelurCoreEngine()
    client = TelurWalletInterface()
    client.start_terminal(core_network)
