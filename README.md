import hashlib
import time
import secrets

class TelurCoreEngine:
    def __init__(self):
        self.ledger_chain = []
        self.vault_balances = {}
        self.max_cap = 50000000.0
        self.circulating_supply = 0.0
        self.base_gas = 5.0
        self.block_subsidy = 50.0
        
        # KONFIGURASI TOKO / AMM POOL
        self.token_price_idr = 10000.0  # Harga 1 TLR = Rp 10.000
        self.shop_address = "tlr_central_liquidity_pool"
        self.shop_fiat_vault = 50000000.0  # Kas awal toko: Rp 50.000.000
        self.vault_balances[self.shop_address] = 5000000.0  # Stok koin di toko: 5 Juta TLR
        self.circulating_supply += 5000000.0
        
        # Inisialisasi Akun Founder Utama
        self.master_key = "priv_" + hashlib.sha256(b"nmara5220@gmail.com-190619").hexdigest()[:32]
        self.genesis_address = "tlr_" + hashlib.sha256(self.master_key.encode()).hexdigest()[:20]
        
        self.vault_balances[self.genesis_address] = 15000000.0
        self.circulating_supply += 15000000.0
        
        # Buat dompet simulasi Rupiah untuk melacak uang fiat pengguna
        self.fiat_balances = {self.genesis_address: 100000000.0} # Dana awal Anda Rp 100 Juta
        
        self.commit_new_block(prev_node="0", payload="Telur Network SECURE CORE with AMM Active", miner_id="NETWORK_PIONEER")

    def generate_keypair(self):
        secret_seed = "priv_" + secrets.token_hex(16)
        public_node = "tlr_" + hashlib.sha256(secret_seed.encode()).hexdigest()[:20]
        self.vault_balances[public_node] = 0.0
        self.fiat_balances[public_node] = 20000000.0  # Pengguna baru diberi modal simulasi Rp 20 Juta
        return secret_seed, public_node

    def execute_shop_buy(self, auth_token, fiat_amount):
        """FITUR BARU: Fungsi untuk membeli TLR dari Toko menggunakan Rupiah"""
        buyer_node = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if buyer_node not in self.vault_balances:
            return False, "ERR_AUTH_FAILED"
            
        if self.fiat_balances.get(buyer_node, 0.0) < fiat_amount:
            return False, "ERR_INSUFFICIENT_FIAT"
            
        tokens_to_receive = fiat_amount / self.token_price_idr
        if self.vault_balances[self.shop_address] < tokens_to_receive:
            return False, "ERR_SHOP_OUT_OF_STOCK"
            
        # Eksekusi Pertukaran
        self.fiat_balances[buyer_node] -= fiat_amount
        self.shop_fiat_vault += fiat_amount
        self.vault_balances[self.shop_address] -= tokens_to_receive
        self.vault_balances[buyer_node] += tokens_to_receive
        
        log = f"SHOP_BUY_{buyer_node[:6]}_VAL_{int(tokens_to_receive)}_TLR"
        self.commit_new_block(prev_node=self.ledger_chain[-1]["node_hash"], payload=log, miner_id=self.shop_address)
        return True, f"SUCCESS_BOUGHT_{tokens_to_receive:,.0f}_TLR"

    def execute_shop_sell(self, auth_token, token_amount):
        """FITUR BARU: Fungsi untuk menjual kembali TLR ke Toko untuk mendapatkan Rupiah"""
        seller_node = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if seller_node not in self.vault_balances:
            return False, "ERR_AUTH_FAILED"
            
        if self.vault_balances[buyer_node if 'buyer_node' in locals() else seller_node] < token_amount:
            return False, "ERR_INSUFFICIENT_TOKENS"
            
        fiat_to_receive = token_amount * self.token_price_idr
        if self.shop_fiat_vault < fiat_to_receive:
            return False, "ERR_SHOP_BANKRUPT"
            
        # Eksekusi Pertukaran
        self.vault_balances[seller_node] -= token_amount
        self.vault_balances[self.shop_address] += token_amount
        self.shop_fiat_vault -= fiat_to_receive
        self.fiat_balances[seller_node] += fiat_to_receive
        
        log = f"SHOP_SELL_{seller_node[:6]}_VAL_{int(token_amount)}_TLR"
        self.commit_new_block(prev_node=self.ledger_chain[-1]["node_hash"], payload=log, miner_id=self.shop_address)
        return True, f"SUCCESS_SOLD_FOR_IDR_{fiat_to_receive:,.0f}"

    def execute_ledger_transfer(self, auth_token, target_node, value, fee_recipient):
        source_node = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if source_node not in self.vault_balances:
            return False, "ERR_AUTH_FAILED"
            
        total_debit = value + self.base_gas
        if self.vault_balances[source_node] < total_debit:
            return False, "ERR_INSUFFICIENT_FUNDS"
            
        if target_node not in self.vault_balances:
            return False, "ERR_INVALID_TARGET"

        self.vault_balances[source_node] -= total_debit
        self.vault_balances[target_node] += value
        self.vault_balances[fee_recipient] = self.vault_balances.get(fee_recipient, 0.0) + self.base_gas
        
        log_summary = f"TXID_{source_node[:6]}_{target_node[:6]}_VAL_{int(value)}"
        
        if self.circulating_supply + self.block_subsidy <= self.max_cap:
            self.vault_balances[fee_recipient] += self.block_subsidy
            self.circulating_supply += self.block_subsidy
            
        self.commit_new_block(prev_node=self.ledger_chain[-1]["node_hash"], payload=log_summary, miner_id=fee_recipient)
        return True, "TX_SUCCESSFULLY_COMMITTED"

    def commit_new_block(self, prev_node, payload, miner_id):
        height = len(self.ledger_chain)
        timestamp = int(time.time())
        raw_data = f"{height}{timestamp}{payload}{prev_node}{miner_id}"
        node_hash = hashlib.sha256(raw_data.encode()).hexdigest()
        
        self.ledger_chain.append({
            "block_height": height,
            "timestamp": timestamp,
            "payload": payload,
            "miner_id": miner_id,
            "prev_node": prev_node,
            "node_hash": node_hash
        })

if __name__ == "__main__":
    node = TelurCoreEngine()
    print("="*50)
    print("TELUR SYSTEM ONLINE (SHOPS AND LIQUIDITY READY)")
    print("="*50)
    print(f"Token Initial Price   : Rp {node.token_price_idr:,.0f} / TLR")
    print(f"Shop Token Liquidity  : {node.vault_balances[node.shop_address]:,} TLR")
    print(f"Shop Cash Reserve     : Rp {node.shop_fiat_vault:,.0f}")
    print("="*50)
    
