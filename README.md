import hashlib
import time

class TelurCoreEngine:
    def __init__(self):
        self.ledger_chain = []
        self.vault_balances = {}
        self.fiat_balances = {}
        self.token_price_idr = 10000.0  # Harga 1 TLR = Rp 10.000
        self.shop_address = "tlr_central_liquidity_pool"
        self.shop_fiat_vault = 50000000.0
        
        # Akun Utama Founder
        self.master_key = "priv_" + hashlib.sha256(b"nmara5220@gmail.com-190619").hexdigest()[:32]
        self.genesis_address = "tlr_" + hashlib.sha256(self.master_key.encode()).hexdigest()[:20]
        
        # Alokasi Saldo Awal
        self.vault_balances[self.genesis_address] = 15000000.0
        self.fiat_balances[self.genesis_address] = 100000000.0
        self.vault_balances[self.shop_address] = 5000000.0
        
        self.commit_new_block("0", "GENESIS_ACTIVE", "SYSTEM")

    def execute_shop_buy(self, auth_token, fiat_amount):
        buyer = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if self.fiat_balances.get(buyer, 0.0) < fiat_amount:
            return False, "ERR_INSUFFICIENT_FIAT"
            
        tokens = fiat_amount / self.token_price_idr
        if self.vault_balances[self.shop_address] < tokens:
            return False, "ERR_OUT_OF_STOCK"
            
        self.fiat_balances[buyer] -= fiat_amount
        self.shop_fiat_vault += fiat_amount
        self.vault_balances[self.shop_address] -= tokens
        self.vault_balances[buyer] = self.vault_balances.get(buyer, 0.0) + tokens
        
        self.commit_new_block(self.ledger_chain[-1]["node_hash"], f"BUY_{int(tokens)}", buyer)
        return True, "BUY_SUCCESS"

    def execute_shop_sell(self, auth_token, token_amount):
        seller = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if self.vault_balances.get(seller, 0.0) < token_amount:
            return False, "ERR_INSUFFICIENT_TOKENS"
            
        fiat = token_amount * self.token_price_idr
        if self.shop_fiat_vault < fiat:
            return False, "ERR_SHOP_BANKRUPT"
            
        self.vault_balances[seller] -= token_amount
        self.vault_balances[self.shop_address] += token_amount
        self.shop_fiat_vault -= fiat
        self.fiat_balances[seller] = self.fiat_balances.get(seller, 0.0) + fiat
        
        self.commit_new_block(self.ledger_chain[-1]["node_hash"], f"SELL_{int(token_amount)}", seller)
        return True, "SELL_SUCCESS"

    def execute_ledger_transfer(self, auth_token, target, value):
        source = "tlr_" + hashlib.sha256(auth_token.encode()).hexdigest()[:20]
        if self.vault_balances.get(source, 0.0) < (value + 5.0):
            return False, "ERR_INSUFFICIENT_FUNDS"
            
        self.vault_balances[source] -= (value + 5.0)
        self.vault_balances[target] = self.vault_balances.get(target, 0.0) + value
        
        self.commit_new_block(self.ledger_chain[-1]["node_hash"], f"SEND_{int(value)}", source)
        return True, "TRANSFER_SUCCESS"

    def commit_new_block(self, prev, payload, miner):
        h = len(self.ledger_chain)
        t = int(time.time())
        raw = f"{h}{t}{payload}{prev}{miner}"
        node_hash = hashlib.sha256(raw.encode()).hexdigest()
        self.ledger_chain.append({"block_height": h, "payload": payload, "node_hash": node_hash})
