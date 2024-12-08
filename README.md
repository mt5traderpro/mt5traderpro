from solana.rpc.api import Client
from solana.keypair import Keypair
from solana.publickey import PublicKey
from solana.transaction import Transaction
from solana.system_program import TransferParams, transfer
from spl.token.constants import TOKEN_PROGRAM_ID
from spl.token.instructions import transfer_checked
from spl.token.client import Token

# Configuration
SOLANA_RPC_URL = "https://api.devnet.solana.com"  # Devnet URL
client = Client(SOLANA_RPC_URL)

def create_wallet():
    """Crée un nouveau portefeuille Solana."""
    wallet = Keypair.generate()
    print("Adresse du portefeuille:", wallet.public_key)
    return wallet

def get_balance(wallet):
    """Récupère le solde du portefeuille en SOL."""
    balance = client.get_balance(wallet.public_key)
    if balance['result']:
        print(f"Solde du portefeuille: {balance['result']['value']} lamports")
    else:
        print("Erreur lors de la récupération du solde.")

def transfer_tokens(wallet, destination_pubkey, amount, token_address):
    """Transfère des jetons SPL d'un portefeuille à un autre."""
    token = Token(client, PublicKey(token_address), TOKEN_PROGRAM_ID, wallet)
    source_account = token.get_accounts(wallet.public_key)['result']['value'][0]['pubkey']
    destination_account = token.get_accounts(PublicKey(destination_pubkey))['result']['value'][0]['pubkey']
    
    # Construire la transaction
    transaction = Transaction()
    transaction.add(
        transfer_checked(
            source=PublicKey(source_account),
            destination=PublicKey(destination_account),
            owner=wallet.public_key,
            amount=amount,
            decimals=token.get_mint_info()['decimals'],
            token_program_id=TOKEN_PROGRAM_ID,
        )
    )

    # Envoyer la transaction
    try:
        response = client.send_transaction(transaction, wallet)
        print("Transaction envoyée:", response)
    except Exception as e:
        print("Erreur lors de l'envoi de la transaction:", str(e))

# Exemple d'utilisation
if __name__ == "__main__":
    wallet = create_wallet()
    get_balance(wallet)

    # Exemple de transfert de jetons SPL
    DESTINATION_PUBLIC_KEY = "AdressePublicKeyDestinataire"  # Remplacez par une vraie adresse
    TOKEN_ADDRESS = "AdresseDuToken"  # Remplacez par l'adresse d'un jeton SPL
    AMOUNT = 1000  # Quantité à envoyer en unités minimales (lamports pour SOL)

    transfer_tokens(wallet, DESTINATION_PUBLIC_KEY, AMOUNT, TOKEN_ADDRESS)
