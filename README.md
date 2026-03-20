# base666666iimport time
from collections import defaultdict, deque
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

WINDOW = 20          # blocks for baseline
SPIKE_MULT = 3       # spike threshold
TOP_TRACK = 100      # track top contracts


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Detecting contract activity spikes...\n")

    last_block = w3.eth.block_number

    # history per contract
    contract_history = defaultdict(lambda: deque(maxlen=WINDOW))

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block > last_block:

                for b in range(last_block + 1, current_block + 1):

                    block = w3.eth.get_block(b, full_transactions=True)

                    block_counts = defaultdict(int)

                    for tx in block.transactions:
                        if tx.to:
                            block_counts[tx.to] += 1

                    # update history + detect spikes
                    for contract, count in block_counts.items():

                        history = contract_history[contract]

                        if len(history) == WINDOW:
                            avg = sum(history) / WINDOW

                            if avg > 0 and count > avg * SPIKE_MULT:
                                print("🚨 CONTRACT SPIKE")
                                print("Block:", b)
                                print("Contract:", contract)
                                print("Tx in block:", count)
                                print("Avg:", avg)
                                print()

                        history.append(count)

                last_block = current_block

            time.sleep(2)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
