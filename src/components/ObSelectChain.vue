<template>
    <div class="obSelectChainBody">
        <div @click.stop="stopPenetrate" class="selectChainContent">
            <div class="topItem">
                <span>Select a Chain</span>
                <div @click="closerButton" style="position: absolute; top: 0; right: 0">
                    <SvgIconThemed style="width: 20px; height: 20px; cursor: pointer" iconName="close" />
                </div>
            </div>
            <div style="width: 100%; position: relative">
                <input type="text" v-model="keyword" class="input" @input="checkKeyWord()"
                    :placeholder="`input search text`" />
                <SvgIconThemed @click="search" class="searchIcon" icon="search" />
            </div>
        </div>
        <div class="list-content-box ob-scrollbar">

            <div class="list-content">
                <template v-if="isExistChainsGroup">
                    <template v-for="(chains, name) of groupChains" >
                        <div class="contentItem title" >{{ toCapitalize(name) }}</div>
                        <div v-for="(item, index) of chains" :key="name + index"
                            @click="getChainInfo(item, index)" class="contentItem">
                            <svg-icon
                                class="logo"
                                :iconName="item.localID"
                                style="margin-right: 1.5rem;"
                            ></svg-icon>
                            <span>{{ item.chain }}</span>
                            <CommLoading v-if="loadingIndex == index" style="left: 1rem; top: 0rem" width="1.5rem"
                                height="1.5rem" />
                        </div>
                    </template>
                    <div class="contentItem title" >{{ toCapitalize('networks') }}</div>
                    <div v-for="(item, index) in newChainData" :key="item.chain + index" @click="getChainInfo(item, index)"
                        class="contentItem">
                        <svg-icon
                            class="logo"
                            :iconName="item.localID"
                            style="margin-right: 1.5rem;"
                        ></svg-icon>
                        <span>{{ item.chain }}</span>
                        <CommLoading v-if="loadingIndex == index" style="left: 1rem; top: 0rem" width="1.5rem"
                            height="1.5rem" />
                    </div>
                </template>
                <template v-else>
                    <div v-for="(item, index) in newChainData" :key="item.chain + index" @click="getChainInfo(item, index)"
                        class="contentItem">
                        <svg-icon
                            class="logo"
                            :iconName="item.localID"
                            style="margin-right: 1.5rem;"
                        ></svg-icon>
                        <span>{{ item.chain }}</span>
                        <CommLoading v-if="loadingIndex == index" style="left: 1rem; top: 0rem" width="1.5rem"
                            height="1.5rem" />
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
import Web3 from 'web3'
import { DydxHelper } from '../util/dydx/dydx_helper'
import { IMXHelper } from '../util/immutablex/imx_helper'
import util from '../util/util.js'
import {customSort} from '../util/index'

import { compatibleGlobalWalletConf } from '../composition/walletsResponsiveData'
import { SvgIconThemed } from './'
import { connectStarkNetWallet } from '../util/constants/starknet/helper.js'
import { web3State } from '../composition/hooks'
import config, { CHAIN_ID } from '../config';

export default {
    name: 'ObSelectChain',
    components: { SvgIconThemed },
    props: {
        ChainData: {
            type: Array,
            default: function () {
                return []
            },
        },
    },
    data() {
        return {
            keyword: '',
            loadingIndex: -1,
        }
    },
    computed: {
        isExistChainsGroup() {
            return !!Object.keys(this.chainsGroup).length
        },
        chainsGroup() {
            return config.chainsGroup || {}
        },
        localIdsInGroup() {
            const localIdsInGroup = Object.values(this.chainsGroup).reduce((localIds, ids) => {
                localIds.push(...ids)
                return localIds
            }, [])
            return localIdsInGroup || []
        },
        transferChainData: function () {
            const newArray = []
            for (let index = 0; index < this.ChainData.length; index++) {
                const item = this.ChainData[index]

                const chainData = {
                    chain: util.chainName(item),
                    localID: item || 1,
                }
                newArray.push(chainData)
            }
            const chainOrderIds = [
                CHAIN_ID.zksync2, CHAIN_ID.zksync2_test, CHAIN_ID.zksync, CHAIN_ID.zksync_test,
                CHAIN_ID.pozkevm, CHAIN_ID.pozkevm_test, CHAIN_ID.po, CHAIN_ID.po_test, CHAIN_ID.mainnet,
                CHAIN_ID.goerli, CHAIN_ID.ar, CHAIN_ID.ar_test, CHAIN_ID.nova, CHAIN_ID.loopring,
                CHAIN_ID.loopring_test, CHAIN_ID.op, CHAIN_ID.op_test, CHAIN_ID.zkspace, CHAIN_ID.zkspace_test,
                CHAIN_ID.imx, CHAIN_ID.imx_test, CHAIN_ID.metis,CHAIN_ID.dydx,CHAIN_ID.dydx_test, CHAIN_ID.boba,
                CHAIN_ID.starknet, CHAIN_ID.starknet_test, CHAIN_ID.bsc, CHAIN_ID.bsc_test
            ]
            return this.orderChainIds(chainOrderIds, newArray)
        },
        newChainData: function () {
            let chains = this.transferChainData.filter(
                    (item) => !this.localIdsInGroup.find(id=>String(id) === String(item.localID))
                )
            if (this.keyword || this.keyword !== '') {
                chains = chains.filter(item=> item.chain.toLowerCase().includes(this.keyword.toLowerCase()))
            }
            const chainOrderIds = [
                CHAIN_ID.zksync2, CHAIN_ID.zksync2_test, CHAIN_ID.zksync, CHAIN_ID.zksync_test,
                CHAIN_ID.pozkevm, CHAIN_ID.pozkevm_test, CHAIN_ID.po, CHAIN_ID.po_test, CHAIN_ID.mainnet,
                CHAIN_ID.goerli, CHAIN_ID.ar, CHAIN_ID.ar_test, CHAIN_ID.nova, CHAIN_ID.loopring,
                CHAIN_ID.loopring_test, CHAIN_ID.op, CHAIN_ID.op_test, CHAIN_ID.zkspace, CHAIN_ID.zkspace_test,
                CHAIN_ID.imx, CHAIN_ID.imx_test, CHAIN_ID.metis,CHAIN_ID.dydx,CHAIN_ID.dydx_test, CHAIN_ID.boba,
                CHAIN_ID.starknet, CHAIN_ID.starknet_test, CHAIN_ID.bsc, CHAIN_ID.bsc_test
            ]
            return customSort(chainOrderIds,chains)
        },
        groupChains:function() {
            const data = {};
            for (const groupName in this.chainsGroup) {
                const chainsIds = this.chainsGroup[groupName]
                let chains = this.transferChainData.filter(
                    (item) => chainsIds.find(chainId=>String(chainId) === String(item.localID))
                )
                if (this.keyword || this.keyword !== '') {
                    chains = chains.filter(item=> item.chain.toLowerCase().includes(this.keyword.toLowerCase()))
                }
                data[groupName] = chains;
            }
            return data;
        }
    },
    watch: {},
    mounted() { },
    methods: {
        getChainsInGroup(chainLocalIds) {
            if (!chainLocalIds) {
                return []
            }
            let chains =  chainLocalIds.map(id=> {
                return this.transferChainData.find(c => id == c.localID);
            });
            //     if (this.keyword || this.keyword !== '') {
            //     chains = chains.filter(item=> item.chain.toLowerCase().includes(this.keyword.toLowerCase()))
            // }
            return chains.filter(row => row && row.localID)
        },
        toCapitalize(str) {
            if (!str) return ''
            return str.charAt(0).toUpperCase() + str.slice(1)
        },
        orderChainIds: function (chainOrderIds, theArray) {
            theArray.sort((chainInfo, nextChainInfo) => {
                return (
                    chainOrderIds.indexOf(chainInfo.localID) -
                    chainOrderIds.indexOf(nextChainInfo.localID)
                )
            })
            return theArray
        },
        closerButton() {
            this.$emit('closeSelect')
        },
        async getChainInfo(e, index) {
            // When chain use stark system
            if (this.isStarkSystem(e.localID)) {
                try {
                    // starknet
                    if (e.localID === CHAIN_ID.starknet || e.localID === CHAIN_ID.starknet_test) {
                        const { starkIsConnected, starkNetAddress } =
                            web3State.starkNet
                        if (!starkIsConnected && !starkNetAddress) {
                            await connectStarkNetWallet()
                            if (
                                !web3State.starkNet.starkIsConnected &&
                                !web3State.starkNet.starkNetAddress
                            ) {
                                return
                            }
                        }
                    }
                    // immutableX
                    if (e.localID === CHAIN_ID.imx || e.localID === CHAIN_ID.imx_test) {
                        this.loadingIndex = index
                        const coinbase =
                            compatibleGlobalWalletConf.value.walletPayload
                                .walletAddress
                        const imxHelper = new IMXHelper(e.localID)
                        coinbase && (await imxHelper.ensureUser(coinbase))
                    }

                    // dydx
                    if (e.localID === CHAIN_ID.dydx || e.localID === CHAIN_ID.dydx_test) {
                        try {
                            this.loadingIndex = index
                            const coinbase =
                                compatibleGlobalWalletConf.value.walletPayload
                                    .walletAddress
                            const dydxHelper = new DydxHelper(
                                e.localID,
                                new Web3(
                                    compatibleGlobalWalletConf.value.walletPayload.provider
                                ),
                                'MetaMask'
                            )
                            await dydxHelper.getDydxClient(coinbase)
                        } catch (error) {
                            console.error(error)
                        }
                    }

                    this.loadingIndex = -1
                } catch (err) {
                    console.log('obselect getChainInfo error', err)
                    this.$notify.error({
                        title: err.message,
                        duration: 3000,
                    })
                    this.loadingIndex = -1
                    return
                }
            }
            this.$emit('getChainInfo', e)
            this.closerButton()
        },
        stopPenetrate(e) {
            e.stopPropagation
        },
        search() { },
        checkKeyWord() { },
        isStarkSystem(chainId) {
            return [CHAIN_ID.starknet, CHAIN_ID.starknet_test, CHAIN_ID.dydx, CHAIN_ID.dydx_test, CHAIN_ID.imx, CHAIN_ID.imx_test].indexOf(chainId) > -1
        },
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.app {
    .obSelectChainBody {
        width: 320px;
        height: 372px;
    }
}

.app-mobile {
    .obSelectChainBody {
        width: calc(100% - 30px);
        max-height: 90vh;
        height: 372px;
    }
}

.obSelectChainBody {
    position: relative;
    margin: 4.2rem auto;
    // height: calc(
    //   100vh - 8.4rem - var(--top-nav-height) - var(--bottom-nav-height)
    // );
    height: calc(100% - 8.4rem - var(--top-nav-height) - var(--bottom-nav-height));
    border-radius: 20px;
    padding: 20px 0;

    .selectChainContent {
        // margin: 1rem 1.5rem;
        position: relative;
        padding: 0 20px;

        .topItem {
            width: 100%;
            height: 2rem;
            font-size: 2rem;
            font-weight: bold;
            line-height: 2rem;
            display: flex;
            // justify-content: space-between;
            justify-content: center;
            padding: 0 1rem;
            margin-bottom: 18px;
            position: relative;
        }

        .input {
            position: relative;
            border-radius: 2rem;
            margin-bottom: 10px;
            height: 4rem;
            width: 100%;
            outline: none;
            font-size: 1.4rem;
            padding: 10px;
            padding-left: 48px;
            border: none;
        }

        input::placeholder {
            font-size: 1.4rem;
            font-family: 'Inter Regular';
        }

        .searchIcon {
            position: absolute;
            left: 20px;
            top: 10px;
        }
    }

    .list-content-box {
        overflow-y: scroll;
        height: calc(100% - 90px);

        .title {
            font-size: 13px;
            text-align: start;
            font-family: inherit;
            font-weight: 600;
            color: #696969;
        }
    }

    .contentItem {
        width: 100%;
        align-items: center;
        display: flex;
        position: relative;
        padding: 10px 30px;
        cursor: pointer;
        font-weight: 700;
        font-size: 16px;
        line-height: 24px;

        .logo {
            width: 2.4rem;
            height: 2.4rem;
            border-radius: 50%;
            background: rgba($color: #000000, $alpha: 0.05);
            padding: 0.2rem;
        }

        .right {
            text-align: right;
            position: absolute;
            right: 0.5rem;
        }
    }
}

// .ant-input-affix-wrapper >>> .ant-input {
//   background-color: transparent;
//   border: 0;
// }

::v-deep .ant-input {
    background-color: transparent;
    border: 0;
    outline: none;
    color: var(--default-black);
    font-size: 1.4rem;
    height: 100%;
}

::v-deep .ant-input-affix-wrapper {
    border: none;
    background: transparent;
    outline: none;
    box-shadow: none;
}
</style>
