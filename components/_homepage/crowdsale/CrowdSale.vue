<template>
    <div class="crowd-sale">
        <div class="crowd-sale__content">
            <div
                v-if="
                    step ==
                    useRuntimeConfig().public.CROWDSALE_STEPS.CONNECT_WALLET
                "
                class="crowd-sale__wrapper"
            >
                <h3 class="crowd-sale__title">Presale Stage #{{ tier }}</h3>
                <p class="crowd-sale__desc">
                    Ends <strong>{{ endDate }}</strong> or when full.
                </p>
                <p class="crowd-sale__token-price">1 EOBO = ${{ price }}</p>
                <div class="crowd-sale__progressbar">
                    <ProgressBar
                        :percentage="soldPercentage"
                        :is-black="true"
                    />
                </div>
                <p class="crowd-sale__token-sold">
                    {{ formatNumber(totalSold) }} /
                    {{ formatNumber(totalSale) }} EOBO sold
                </p>
                <p class="crowd-sale__sub-desc">
                    Please connect an ERC-20 wallet to participate.
                </p>
                <div class="crowd-sale__button-group">
                    <button class="eobo-btn connect" @click="connectWallet">
                        <span>
                            {{
                                connectingWallet
                                    ? "Connecting..."
                                    : "Connect Your Wallet"
                            }}
                        </span>
                    </button>
                </div>
                <p v-if="nextPrice > 0" class="crowd-sale__sub-desc">
                    Be early! Next round: 1 EOBO = ${{ nextPrice }}
                </p>
            </div>
            <div
                v-if="
                    step ===
                    useRuntimeConfig().public.CROWDSALE_STEPS.SELECT_CURRENCY
                "
                class="crowd-sale__wrapper"
            >
                <h3 class="crowd-sale__title">Presale Stage #{{ tier }}</h3>
                <p class="crowd-sale__desc">
                    Ends when full, or <span>{{ endDate }}</span>
                </p>
                <div class="crowd-sale__progressbar">
                    <ProgressBar
                        :percentage="soldPercentage"
                        :is-black="true"
                    />
                </div>
                <p class="crowd-sale__token-sold">
                    ${{ formatNumber(totalSold * price) }} / ${{
                        formatNumber(totalSale * price)
                    }}
                    raised
                </p>
                <p class="crowd-sale__token-price">1 EOBO = ${{ price }}</p>
                <p class="crowd-sale__sub-desc">
                    Wallet connected! Buy EOBO allocation with:
                </p>
                <div class="crowd-sale__button-group">
                    <button
                        class="eobo-btn currency"
                        @click="
                            selectCurrency(
                                useRuntimeConfig().public.CROWDSALE_CURRENCIES
                                    .ETH
                            )
                        "
                    >
                        <img
                            :src="getCurrencyIcon('eth')"
                            width="16"
                            height="16"
                        />&nbsp; <span>ETH</span>
                    </button>
                    <button
                        class="eobo-btn currency"
                        @click="
                            selectCurrency(
                                useRuntimeConfig().public.CROWDSALE_CURRENCIES
                                    .USDC
                            )
                        "
                    >
                        <img
                            :src="getCurrencyIcon('usdc')"
                            width="16"
                            height="16"
                        />&nbsp; <span>USDC</span>
                    </button>
                    <button
                        class="eobo-btn currency"
                        @click="
                            selectCurrency(
                                useRuntimeConfig().public.CROWDSALE_CURRENCIES
                                    .USDT
                            )
                        "
                    >
                        <img
                            :src="getCurrencyIcon('usdt')"
                            width="16"
                            height="16"
                        />&nbsp; <span>USDT</span>
                    </button>
                    <button
                        class="eobo-btn currency"
                        @click="
                            selectCurrency(
                                useRuntimeConfig().public.CROWDSALE_CURRENCIES
                                    .DAI
                            )
                        "
                    >
                        <img
                            :src="getCurrencyIcon('dai')"
                            width="16"
                            height="16"
                        />&nbsp; <span>DAI</span>
                    </button>
                </div>
                <p v-if="nextPrice > 0" class="crowd-sale__sub-desc">
                    Be early! Next round: 1 EOBO = ${{ nextPrice }}
                </p>
            </div>
            <div
                v-if="
                    step === useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN
                "
                class="crowd-sale__wrapper"
            >
                <div class="crowd-sale__button-back" @click="back">
                    <img src="/assets/images/icons/arrow-left-black.png" />
                </div>
                <h3 class="crowd-sale__title">
                    Buy EOBO with {{ buyWithCurrency }}
                </h3>
                <p class="crowd-sale__desc" :class="error ? 'error' : null">
                    {{
                        error
                            ? error
                            : `Make sure you have enough ${buyWithCurrency} sitting in your wallet for the allocation.`
                    }}
                </p>
                <div class="crowd-sale__input-wrapper">
                    <input
                        v-model="amount"
                        onkeypress="return event.charCode >= 48 && event.charCode <= 57"
                        class="crowd-sale__input"
                        maxlength="8"
                        placeholder="Enter the amount of EOBO..."
                    />
                    <img src="/assets/images/coins/eobo.svg" alt="Rewarded" />
                </div>
                <p class="crowd-sale__token-price">
                    {{ amount || "0" }} EOBO =
                    {{
                        currency ===
                        useRuntimeConfig().public.CROWDSALE_CURRENCIES.ETH
                            ? payment.toFixed(4)
                            : payment
                    }}
                    {{ buyWithCurrency }}
                </p>
                <div class="crowd-sale__checkbox">
                    <input
                        id="agreement"
                        v-model="agreed"
                        type="checkbox"
                        name="agreement"
                    />
                    <label for="agreement">
                        I agree to the
                        <a href="/token-sale-agreement" target="_blank"
                            >token sale agreement</a
                        >
                    </label>
                </div>
                <div class="crowd-sale__button-group">
                    <button
                        class="eobo-btn buy"
                        :disabled="!agreed || !payment"
                        @click="buyToken"
                    >
                        <span>
                            <img
                                :src="getCurrencyIcon()"
                                width="24"
                                height="24"
                            />
                            Pay
                            {{
                                currency ===
                                useRuntimeConfig().public.CROWDSALE_CURRENCIES
                                    .ETH
                                    ? payment.toFixed(4)
                                    : payment
                            }}
                            {{ buyWithCurrency }}
                        </span>
                    </button>
                </div>
                <p class="crowd-sale__sub-desc">
                    You will be able to claim your tokens on launch.
                </p>
            </div>
            <div
                v-if="
                    step ===
                    useRuntimeConfig().public.CROWDSALE_STEPS.BOUGHT_TOKEN
                "
                class="crowd-sale__wrapper"
            >
                <div
                    v-if="transaction"
                    class="crowd-sale__button-back"
                    @click="back"
                >
                    <img src="/assets/images/icons/arrow-left-black.png" />
                </div>
                <h3 class="crowd-sale__title">
                    {{ transaction ? "Success!" : "Processing..." }}
                </h3>
                <p class="crowd-sale__desc">
                    Thank you for supporting our project.
                </p>
                <img
                    src="/assets/images/coins/eobo.svg"
                    class="logo-image"
                    :class="transaction ? null : 'shaking'"
                    alt="Rewarded"
                />
                <p class="crowd-sale__token-price">{{ amount }} EOBO</p>
                <p class="crowd-sale__sub-desc">
                    You will be able to start claiming your tokens on launch.
                    Join our communities to stay updated!
                </p>
                <a
                    v-if="transaction"
                    class="crowd-sale__tx-link"
                    :href="transactionLink"
                    target="_blank"
                    >View TX on Etherscan</a
                >
            </div>
        </div>
        <p v-if="tokensBought > 0" class="crowd-sale__bottom-allocation">
            Your current allocation: {{ formatNumber(tokensBought) }} EOBO
        </p>
        <p class="crowd-sale__bottom-title">Token launch in Q3 2024!</p>
    </div>
</template>

<script>
import ProgressBar from "@/components/_homepage/crowdsale/ProgressBar";
import abiCrowdSaleContract from "~/app/abi/abiCrowdSaleContract";
import abiERC20 from "~/app/abi/abiERC20";
import {
    createWeb3Modal,
    useWeb3Modal,
    defaultWagmiConfig,
} from "@web3modal/wagmi/vue";
import {
    createConfig,
    http,
    watchAccount,
    readContract,
    switchChain,
    disconnect,
    reconnect,
    getChainId,
} from "@wagmi/core";
import { mainnet, goerli } from "@wagmi/core/chains";

export default {
    name: "CrowdSale",
    components: {
        ProgressBar,
    },
    setup() {
        const chains = [mainnet, goerli];
        const projectId = useRuntimeConfig().public.WAGMI_PROJECT_ID;
        const metadata = {
            name: "eobo",
            description: "Welcome to the $EOBO Crowdsale!",
            url: useRuntimeConfig().public.BASE_APP_URL,
            icons: ["https://eobo.tech/eobo-icon-200.pngs"],
        };
        const wagmiConfig = defaultWagmiConfig({
            chains,
            projectId,
            metadata,
            transports: {
                [mainnet.id]: http(
                    useRuntimeConfig().public.INFURA_MAINNET_ENDPOINT
                ),
                [goerli.id]: http(
                    useRuntimeConfig().public.INFURA_TESTNET_ENDPOINT
                ),
            },
        });
        createWeb3Modal({
            wagmiConfig: wagmiConfig,
            projectId,
            chains,
            enableAnalytics: true,
        });
        const modal = useWeb3Modal();
        return {
            modal,
            wagmiConfig,
        };
    },
    data() {
        return {
            step: useRuntimeConfig().public.CROWDSALE_STEPS.CONNECT_WALLET,
            currency: useRuntimeConfig().public.CROWDSALE_CURRENCIES.ETH,
            soldPercentage: 0,
            agreed: false,
            connectingWallet: false,
            connectedWallet: "",
            tier: 1,
            web3: null,
            saleContract: null,
            totalSale: 0,
            totalSold: 0,
            inSale: 0,
            price: 0,
            startTimestamp: 0,
            endTimestamp: 0,
            endDate: "",
            nextPrice: 0,
            amount: "",
            payment: 0,
            tokensBought: 0,
            transaction: null,
            error: "",
        };
    },
    computed: {
        buyWithCurrency() {
            switch (this.currency) {
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.ETH:
                    return "ETH";
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDT:
                    return "USDT";
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDC:
                    return "USDC";
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.DAI:
                    return "DAI";
            }
            return "";
        },
        transactionLink() {
            if (this.transaction) {
                return `${useRuntimeConfig().public.ETHERSCAN_BASE_URL}/tx/${
                    this.transaction.transactionHash
                }`;
            }
            return "/";
        },
    },
    watch: {
        amount(newAmount) {
            if (newAmount.length > 0) {
                const numAmount = Number(newAmount);
                if (
                    numAmount >=
                        useRuntimeConfig().public.CROWDSALE_MIN_AMOUNT &&
                    numAmount <= useRuntimeConfig().public.CROWDSALE_MAX_AMOUNT
                ) {
                    this.calculatePayment(newAmount);
                } else {
                    this.payment = 0;
                }
            } else {
                this.payment = 0;
            }
        },
    },
    mounted() {
        this.getSaleContractStatus();
        const unwatch = watchAccount(this.wagmiConfig, {
            onChange(account) {
                console.log("account---->",account);
                if (!account.isConnected) {
                    this.step =
                        useRuntimeConfig().public.CROWDSALE_STEPS.CONNECT_WALLET;
                } else {
                    this.loggedInWithWallet();
                }
            },
        });
        unwatch();
    },
    methods: {
        getCurrencyIcon(c) {
            if (c === undefined) {
                switch (this.currency) {
                    case useRuntimeConfig().public.CROWDSALE_CURRENCIES.ETH:
                        return "/assets/images/coins/eth.svg";
                    case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDT:
                        return "/assets/images/coins/usdt.svg";
                    case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDC:
                        return "/assets/images/coins/usdc.svg";
                    case useRuntimeConfig().public.CROWDSALE_CURRENCIES.DAI:
                        return "/assets/images/coins/dai.svg";
                }
            } else {
                return "/assets/images/coins/" + c.toLowerCase() + ".svg";
            }
            return null;
        },
        formatNumber(number) {
            return number.toLocaleString("en-UK", {
                maximumSignificantDigits: 3,
            });
        },
        async connectWallet() {
            this.connectingWallet = true;
            this.modal.open();
        },
        async loggedInWithWallet() {
            const { chainId } = getChainId();
            if (chainId != useRuntimeConfig().public.CROWDSALE_NETWORK_ID) {
                switchChain({
                    chainId: useRuntimeConfig().public.CROWDSALE_NETWORK_ID,
                });
            }
            this.step =
                useRuntimeConfig().public.CROWDSALE_STEPS.SELECT_CURRENCY;
            this.getUserContractStatus();
            this.connectingWallet = false;
        },
        selectCurrency(currency) {
            this.step = useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN;
            this.currency = currency;
            this.amount = "";
            this.payment = 0;
            this.error = "";
        },
        async buyToken() {
            try {
                if (this.agreed && window.ethereum) {
                    const provider = new ethers.BrowserProvider(
                        window.ethereum
                    );
                    const signer = await provider.getSigner();
                    const saleContract = new ethers.Contract(
                        useRuntimeConfig().public.CROWDSALE_CONTRACT_ADDRESS,
                        abiCrowdSaleContract,
                        signer
                    );

                    this.step =
                        useRuntimeConfig().public.CROWDSALE_STEPS.BOUGHT_TOKEN;
                    this.transaction = null;
                    this.error = "";

                    if (
                        new Date().getTime() < this.startTimestamp * 1000 ||
                        new Date().getTime() > this.endTimestamp * 1000
                    ) {
                        this.error =
                            "Sale time is over! Thank you for your interest.";
                        this.step =
                            useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN;
                        return;
                    }

                    if (Number(this.amount) > this.inSale) {
                        this.error = "There are insufficient tokens in sale.";
                        this.step =
                            useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN;
                        return;
                    }

                    switch (this.currency) {
                        case useRuntimeConfig().public.CROWDSALE_CURRENCIES
                            .ETH: {
                            const tx = await saleContract.buyWithETH(
                                this.tier,
                                this.amount,
                                { value: ethers.parseEther(`${this.payment}`) }
                            );
                            this.transaction = await tx.wait();
                            break;
                        }
                        case useRuntimeConfig().public.CROWDSALE_CURRENCIES
                            .USDT: {
                            const usdtContract = new ethers.Contract(
                                useRuntimeConfig().public.USDT_CONTRACT_ADDRESS,
                                abiERC20,
                                signer
                            );
                            let tx = await usdtContract.approve(
                                useRuntimeConfig().public
                                    .CROWDSALE_CONTRACT_ADDRESS,
                                this.payment * 10 ** 6
                            );
                            await tx.wait();
                            tx = await saleContract.buyWithUSDT(
                                this.tier,
                                this.amount
                            );
                            this.transaction = await tx.wait();
                            break;
                        }
                        case useRuntimeConfig().public.CROWDSALE_CURRENCIES
                            .USDC: {
                            const usdcContract = new ethers.Contract(
                                useRuntimeConfig().public.USDC_CONTRACT_ADDRESS,
                                abiERC20,
                                signer
                            );
                            let tx = await usdcContract.approve(
                                useRuntimeConfig().public
                                    .CROWDSALE_CONTRACT_ADDRESS,
                                this.payment * 10 ** 6
                            );
                            await tx.wait();
                            tx = await saleContract.buyWithUSDC(
                                this.tier,
                                this.amount
                            );
                            this.transaction = await tx.wait();
                            break;
                        }
                        case useRuntimeConfig().public.CROWDSALE_CURRENCIES
                            .DAI: {
                            const daiContract = new ethers.Contract(
                                useRuntimeConfig().public.DAI_CONTRACT_ADDRESS,
                                abiERC20,
                                signer
                            );
                            let tx = await daiContract.approve(
                                useRuntimeConfig().public
                                    .CROWDSALE_CONTRACT_ADDRESS,
                                ethers.parseEther(`${this.payment}`)
                            );
                            await tx.wait();
                            tx = await saleContract.buyWithDAI(
                                this.tier,
                                this.amount
                            );
                            this.transaction = await tx.wait();
                            break;
                        }
                    }
                    this.getSaleContractStatus();
                }
            } catch (e) {
                console.log(e);
                this.step = useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN;
                this.error =
                    "Transaction failed. Make sure you have enough funds in your wallet and try again!";
                this.transaction = null;
            }
        },
        back() {
            if (
                this.step ===
                useRuntimeConfig().public.CROWDSALE_STEPS.BOUGHT_TOKEN
            ) {
                this.step = useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN;
            } else if (
                this.step ===
                useRuntimeConfig().public.CROWDSALE_STEPS.BUY_TOKEN
            ) {
                this.step =
                    useRuntimeConfig().public.CROWDSALE_STEPS.SELECT_CURRENCY;
            }
        },
        async getSaleContractStatus() {
            try {
                this.tier = Number(useRuntimeConfig().public.CROWDSALE_TIER);
                console.log(this.tier, this.wagmiConfig);
                console.log(abiCrowdSaleContract);
                console.log(
                    this.tier,
                    useRuntimeConfig().public.CROWDSALE_NETWORK_ID
                );
                const status = await readContract(this.wagmiConfig, {
                    abi: abiCrowdSaleContract,
                    address:
                        useRuntimeConfig().public.CROWDSALE_CONTRACT_ADDRESS,
                    chainId: Number(
                        useRuntimeConfig().public.CROWDSALE_NETWORK_ID
                    ),
                    functionName: "presale",
                    args: [this.tier],
                });

                console.log("status", status);

                const dayjs = useDayjs();

                this.totalSale = Number(status.tokensToSell);
                this.totalSold =
                    Number(status.tokensToSell) - Number(status.inSale);
                this.inSale = Number(status.inSale);
                this.price = Number(status.price) / 10 ** 18;
                this.soldPercentage = (this.totalSold / this.totalSale) * 100;
                this.startTimestamp = Number(status.startTime);
                this.endTimestamp = Number(status.endTime);
                this.endDate = dayjs(this.endTimestamp * 1000).format(
                    "DD MMMM YYYY"
                );

                if (this.tier < 3) {
                    const status = await readContract(this.wagmiConfig, {
                        abi: abiCrowdSaleContract,
                        address:
                            useRuntimeConfig().public
                                .CROWDSALE_CONTRACT_ADDRESS,
                        chainId: Number(
                            useRuntimeConfig().public.CROWDSALE_NETWORK_ID
                        ),
                        functionName: "presale",
                        args: [this.tier + 1],
                    });
                    this.nextPrice = Number(status.price) / 10 ** 18;
                }
            } catch (e) {
                console.log(e);
            }
        },
        async getUserContractStatus() {
            try {
                const userVesting = await readContract(this.wagmiConfig, {
                    abi: abiCrowdSaleContract,
                    address:
                        useRuntimeConfig().public.CROWDSALE_CONTRACT_ADDRESS,
                    chainId: Number(
                        useRuntimeConfig().public.CROWDSALE_NETWORK_ID
                    ),
                    functionName: "userVesting",
                    args: [this.connectedWallet, this.tier],
                });

                this.tokensBought = Number(userVesting.totalAmount) / 10 ** 18;
            } catch (e) {
                console.log(e);
            }
        },
        async calculatePayment(amount) {
            if (!this.saleContract) {
                return 0;
            }
            let payment = 0;
            switch (this.currency) {
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.ETH:
                    payment = await this.saleContract.methods
                        .ethBuyHelper(this.tier, Number(amount))
                        .call();
                    payment = Number(payment) / 10 ** 18 + 0.00001;
                    break;
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.DAI:
                    payment = await this.saleContract.methods
                        .daiBuyHelper(this.tier, Number(amount))
                        .call();
                    payment = Number(payment) / 10 ** 18;
                    break;
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDT:
                case useRuntimeConfig().public.CROWDSALE_CURRENCIES.USDC:
                    payment = await this.saleContract.methods
                        .stablecoinBuyHelper(this.tier, Number(amount))
                        .call();
                    payment = Number(payment) / 10 ** 6;
            }
            this.payment = payment;
            return payment;
        },
    },
};
</script>

<style scoped lang="scss">
.crowd-sale {
    animation: fadeIn 1s;
    max-width: 420px;
    min-height: 480px;
    background: #ffffffaa;
    backdrop-filter: blur(15px);
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0px 4px 4px rgba(19, 29, 60, 0.05);

    @media screen and (max-width: 900px) {
        position: relative;
        min-width: 100%;
        transform: translate(0, 0);
        margin-top: 60px;
    }

    &__wrapper {
        width: 100%;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;

        .logo-image {
            width: 80px;
            height: 80px;
            object-fit: fill;
            margin: 10px;
            &.shaking {
                animation: shaking 2s ease infinite;
            }
        }
    }

    &__title {
        font-weight: 600;
        font-size: 32px;
        font-family: $title-font;
        text-transform: uppercase;
        color: black;
        margin: 10px;
        text-align: center;
    }

    &__desc {
        font-weight: 400;
        font-size: 14px;
        color: #7b786f;
        margin: 10px;
        text-align: center;
        &.error {
            color: #cc0000;
        }
    }

    &__token-price {
        font-weight: 600;
        font-size: 32px;
        color: #ff002e;
        margin: 10px;
        text-align: center;
        font-family: Antonio, serif;
    }

    &__token-sold {
        font-weight: 400;
        font-size: 16px;
        color: black;
        margin: 10px;
        text-align: center;
    }

    &__sub-desc {
        font-weight: 400;
        font-size: 14px;
        color: #7b786f;
        margin: 10px;
        text-align: center;
    }

    &__bottom-title,
    &__bottom-allocation {
        font-weight: 400;
        font-size: 14px;
        color: black;
        margin: 10px;
        text-align: center;
    }

    &__bottom-allocation {
        font-weight: 600;
        color: #47c2ff;
    }

    &__button-back {
        position: absolute;
        left: 20px;
        top: 20px;
        cursor: pointer;

        img {
            width: 28px;
            height: 20px;
            object-fit: fill;
        }
    }

    &__tx-link {
        text-decoration: underline;
        margin: 10px;
    }

    &__input-wrapper {
        width: 90%;
        height: 60px;
        padding: 0 20px;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        background: #aaaaaa22;
        border-radius: 8px;
        margin: 10px;

        input {
            flex-grow: 1;
            font-weight: 400;
            font-size: 16px;
            color: black;
            text-align: left;
            background: transparent;
            outline: none;
            border: none;

            &:focus {
                outline: none;
                border: none;
            }
        }

        img {
            width: 30px;
            height: 30px;
            object-fit: fill;
        }
    }

    &__progressbar {
        width: 100%;
        margin: 10px;
        position: relative;

        .progress-bar {
            background: white;
        }
    }

    &__button-group {
        width: 100%;
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        margin: 10px;

        & > * + * {
            margin-left: 0.25rem;
        }
    }

    &__checkbox {
        color: black;
        margin: 10px;

        input {
            accent-color: black;
            font-size: 16px;
        }

        label {
            cursor: pointer;
            font-size: 16px;
            margin-left: 5px;

            a {
                text-decoration: underline;
            }
        }
    }

    .eobo-btn {
        flex: 1;
        max-width: 260px;
        background-color: transparent;
        border-color: #040315;
        color: #040315;

        &:hover {
            background-color: #fff;
            transition: 0.5s;
        }

        &.connect {
            padding: 6px 12px;

            span {
                font-weight: 700;
                font-size: 16px;
                color: black;
            }
        }

        &.buy {
            padding: 6px 12px;
            display: flex;
            justify-content: space-around;

            span {
                align-items: center;
                font-weight: 700;
                font-size: 16px;
                color: black;
                display: flex;

                img {
                    margin-right: 12px;
                }
            }

            &:disabled {
                opacity: 0.2;
            }
        }

        &.currency {
            max-width: 80px;
            display: flex;
            flex-direction: row;
            justify-content: center;
            align-items: center;
            padding: 6px 12px;

            span {
                font-weight: 700;
                font-size: 14px;
                color: black;
            }
        }
    }
}

@keyframes fadeIn {
    0% {
        opacity: 0;
    }

    100% {
        opacity: 1;
    }
}

@keyframes shaking {
    0%,
    100% {
        transform: translateX(0);
    }
    10%,
    30%,
    50%,
    70%,
    90% {
        transform: translateX(-10px);
    }
    20%,
    40%,
    60%,
    80% {
        transform: translateX(10px);
    }
}
</style>
