import hashlib
import secrets

class TelurWalletInterface:
    def __init__(self):
        self.session_key = None
        self.session_address = None

    def start_terminal(self, engine_instance):
        while True:
            print("\n" + "="*45)
            print("🚀  TELUR NETWORK TERMINAL CLIENT v1.0  🚀")
            print("="*45)
            if self.session_address:
                print(f"STATUS: ACTIVE SESSION")
                print(f"NODE ADDRESS: {self.session_address}")
            else:
                print("STATUS: DISCONNECTED")
            print("-"*45)
            print("1. Initialize New Wallet Keypair")
            print("2. Authenticate with Existing Secret Key")
            print("3. Query Node Balance")
            print("4. Broadcast Transaction")
            print("5. Terminate Terminal")
            print("="*45)
            
            cmd = input("Enter command (1-5): ").strip()

            if cmd == "1":
                self.init_keypair(engine_instance)
            elif cmd == "2":
                self.auth_session()
            elif cmd == "3":
                self.query_balance(engine_instance)
            elif cmd == "4":
                self.broadcast_tx(engine_instance)
            elif cmd == "5":
                print("\n[INFO] Terminal session terminated.")
                break
            else:
                print("[WARN] Invalid command sequence.")

    def init_keypair(self, engine):
        secret_seed = "priv_" + secrets.token_hex(16)
        public_node = "tlr_" + hashlib.sha256(secret_seed.encode()).hexdigest()[:20]
        
        engine.vault_balances[public_node] = 0.0
        self.session_key = secret_seed
        self.session_address = public_node
        
        print("\n[SUCCESS] NEW KEYPAIR GENERATED")
        print(f"Public Address : {public_node}")
        print(f"Secret Key     : {secret_seed}")
        print("[CRITICAL] Backup your secret key. Lost keys cannot be recovered.")

    def auth_session(self):
        token_input = input("\nEnter Secret Key (priv_...): ").strip()
        if not token_input.startswith("priv_") or len(token_input) < 20:
            print("[ERROR] Authentication token malformed.")
            return
            
        self.session_address = "tlr_" + hashlib.sha256(token_input.encode()).hexdigest()[:20]
        self.session_key = token_input
        print(f"\n[SUCCESS] Authenticated. Active node: {self.session_address}")

    def query_balance(self, engine):
        if not self.session_address:
            print("[ERROR] No active session found. Please authenticate first.")
            return
            
        balance = engine.vault_balances.get(self.session_address, 0.0)
        print(f"\n[BALANCE] Verified Ledger Balance: {balance:,} TLR")

    def broadcast_tx(self, engine):
        if not self.session_key:
            print("[ERROR] Transaction unsigned. Authenticate session.")
            return
            
        target = input("\nEnter Destination Address (tlr_...): ").strip()
        try:
            amount = float(input("Enter Value to Transfer: "))
        except ValueError:
            print("[ERROR] Value must be numerical.")
            return
            
        miner_node = "tlr_network_central_pool"
        print("\n[PROCESSING] Signing payload and broadcasting to ledger...")
        
        status, log_msg = engine.execute_ledger_transfer(
            self.session_key, target, amount, miner_node
        )
        print(f"[NODE LOG] {log_msg}")

if __name__ == "__main__":
    from coin_telur import TelurCoreEngine
    
    core_network = TelurCoreEngine()
    client = TelurWalletInterface()
    client.start_terminal(core_network)
    
