<template>
<div class="accele-raytor container">
    <div class="banner">
      <div class="text-part">
        <div class="title">AcceleRaytor</div>
        <div class="description">
          A launchpad for new projects to raise capital and for the community to participate in new Solana project
          launches.
        </div>
      </div>
    </div> 

    <div>
      <Input size="large" placeholder="Search by Pool name, Token symbol, Address">
        <Icon slot="prefix" type="search" />
      </Input>
      <div class="filter">
        <div class="group">
          <h5>Access</h5>
          <RadioGroup v-model="filter.access">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="ray">RAY</RadioButton>
            <RadioButton value="community">Community</RadioButton>
            <RadioButton value="private">Private</RadioButton>
          </RadioGroup>
        </div>
        <div class="group">
          <h5>Status</h5>
          <RadioGroup v-model="filter.status">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="open">Open</RadioButton>
            <RadioButton value="upcoming">Upcoming</RadioButton>
            <RadioButton value="ended">Ended</RadioButton>
          </RadioGroup>
        </div>
        <div class="group">
          <h5>My pools</h5>
          <RadioGroup v-model="filter.mine">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="joined">Joined</RadioButton>
          </RadioGroup>
        </div>
      </div>
      <Table
        :columns="columns"
        :data-source="pools"
        :pagination="false"
        row-key="idoId"
        :custom-row="
          (record) => {
            return {
              on: {
                click: () => {
                  $router.push(`/acceleraytor/${record.idoId}/`)
                }
              }
            }
          }
        "
      >
        <span slot="name" slot-scope="base" class="icon">
          <CoinIcon :mint-address="base.mintAddress" />
          <span> {{ base.symbol }}</span>
        </span>
        <span slot="price" slot-scope="price, pool"> {{ price.toEther() }} {{ pool.quote.symbol }} </span>
        <span slot="access" slot-scope="isRayPool, pool" class="access">
          <span v-if="pool.isPrivate" class="community">
            <span>Private Pool</span>
          </span>
          <span v-else-if="pool.isRayPool" class="ray">
            <span>{{ `RAY ${pool.version === 3 ? '' : pool.info.minStakeLimit.format()} Pool` }}</span>
          </span>
          <span v-else class="community"><span>Community Pool</span></span>
        </span>
        <span slot="allocation" slot-scope="info, pool">
          {{ pool.version === 3 ? 'Lottery' : pool.info.maxDepositLimit.format() + pool.quote.symbol }}
        </span>
        <span slot="raise" slot-scope="raise, pool"> {{ raise.format() }} {{ pool.base.symbol }} </span>
        <span slot="filled" slot-scope="info, pool">
          {{
            (pool.version === 3
              ? (pool.info.currentLotteryNumber / pool.info.totalWinLotteryLimit) * 100
              : parseInt(
                  getBigNumber(
                    info.quoteTokenDeposited
                      .toEther()
                      .dividedBy(pool.raise.toEther().multipliedBy(pool.price.toEther()))
                      .multipliedBy(100)
                  )
                )) + '%'
          }}
        </span>
        <span slot="status" slot-scope="info, pool" class="status">
          <span :class="pool.status">{{ pool.status }}</span>
        </span>
      </Table>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'

import { Input, Icon, Radio, Table } from 'ant-design-vue'
import { filter } from 'lodash-es'

import { getUnixTs } from '@/utils'
import { IdoPool } from '@/utils/ido'
import { getBigNumber } from '@/utils/layouts'

const RadioGroup = Radio.Group
const RadioButton = Radio.Button

@Component({
  head: {
    title: 'AcceleRaytor'
  },

  components: {
    Input,
    Icon,
    RadioGroup,
    RadioButton,
    Table
  },

  async asyncData({ $accessor }) {
    if (!$accessor.ido.initialized) {
      await $accessor.ido.requestInfos()
    }
  }
})
export default class AcceleRaytor extends Vue {
  filter = {
    access: 'all',
    status: 'all',
    mine: 'all'
  }

  pools: IdoPool[] = []

  @Watch('filter', { immediate: true, deep: true })
  onFilterChanged({ access, status }: { access: string; status: string; mine: string }) {
    const rules = {
      info: {}
    } as any

    switch (access) {
      case 'ray': {
        rules.isRayPool = true
        rules.isPrivate = false
        break
      }
      case 'community': {
        rules.isRayPool = false
        rules.isPrivate = false
        break
      }
      case 'private': {
        rules.isPrivate = true
        break
      }
    }
    if (status !== 'all') rules.status = status
    this.pools = filter(this.$accessor.ido.pools, rules)
  }

  columns = [
    {
      title: 'Pool name',
      dataIndex: 'base',
      key: 'name',
      scopedSlots: { customRender: 'name' }
    },
    {
      title: 'Price per token',
      dataIndex: 'price',
      key: 'price',
      scopedSlots: { customRender: 'price' },
      align: 'center'
    },
    {
      title: 'Access',
      dataIndex: 'isRayPool',
      key: 'access',
      scopedSlots: { customRender: 'access' },
      align: 'center'
    },
    {
      title: 'Max Allocation',
      dataIndex: 'allocation',
      key: 'allocation',
      scopedSlots: { customRender: 'allocation' },
      align: 'center'
    },
    {
      title: 'Raise size',
      dataIndex: 'raise',
      key: 'raise',
      scopedSlots: { customRender: 'raise' },
      align: 'center'
    },
    {
      title: 'Filled',
      dataIndex: 'info',
      key: 'filled',
      scopedSlots: { customRender: 'filled' },
      align: 'center'
    },
    {
      title: 'Status',
      dataIndex: 'info',
      key: 'status',
      scopedSlots: { customRender: 'status' },
      align: 'center'
    }
  ]

  getUnixTs = getUnixTs
  getBigNumber = getBigNumber
}
</script>

<style lang="less" scoped>
.accele-raytor.container {
  max-width: 1200px;
}

.accele-raytor {
  .filter {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin-top: 16px;
    padding: 12px;
    background: #4a991d;
    color: #f0f7f1bf;

    .group {
      padding: 12px;
    }
    h5 {
      color: #f1f1f2bf;
    }
  }

  @media (max-width: 850px) {
    .filter {
      display: block;
    }
  }

  .icon {
    display: flex;
    flex-direction: row;
    align-items: center;

    img {
      border-radius: 50%;
      width: 40px;
      height: 40px;
      margin-right: 8px;
    }
  }

  .access {
    .community,
    .ray {
      padding: 5px 1px;
      border-radius: 16px;

      span {
        padding: 4px 14px;
        border-radius: 16px;
      }
    }

    .community {
      background: #39db6f;
    }

    .ray {
      background: linear-gradient(245.22deg, #4fa119 7.97%, #f4f5f8 49.17%, #37fe91 49.17%, #5ac4be 92.1%);

      span {
        background: #3caa10;
        opacity: 0.9;
      }
    }
  }

  .status {
    text-transform: capitalize;

    .upcoming,
    .open,
    .ended {
      padding: 4px 14px;
      border-radius: 16px;
    }

    .upcoming {
      background: rgba(90, 196, 190, 0.2);
      border: 1px solid #5ac4be;
    }

    .open {
      background: rgba(90, 196, 190, 0.2);
      border: 1px solid #5ac4be;
    }

    .ended {
      background: rgba(194, 0, 251, 0.2);
      border: 1px solid #c200fb;
    }
  }
}
</style>

<style lang="less">
@import '../../styles/variables';

.banner {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  align-items: center;
  grid-gap: 80px;
  gap: 80px;
  border-radius: 28px;
  padding: 32px 64px;
  margin: 36px -32px 64px;
  background-color: #23660e;
  .text-part {
    .title {
      font-weight: 600;
      font-size: 48px;
    }
    .description {
      font-weight: 300;
      font-size: 20px;
      color: #f0f1f3;
    }
  }
  .image-part {
    display: flex;
    flex-direction: column;
    align-items: center;
    .decorator-text {
      font-size: 24px;
      margin-bottom: 14px;
      text-align: center;
      .secondary-decorator-text {
        font-size: 1rem;
      }
      .decorator-link {
        margin-top: 8px;
        font-size: 1rem;
      }
    }
    img {
      width: 540px;
      object-fit: contain;
      border-radius: 20px;
    }
  }
}

.has-error {
  input {
    border-color: #f5222d;

    &:active,
    &:focus,
    &:hover {
      border-color: #f5222d !important;
    }
  }
}

.accele-raytor {
  .ant-input-affix-wrapper {
    input {
      background-color: @bg-color;
    }
  }

  .ant-radio-button-wrapper {
    border: 1px solid transparent;
    box-shadow: none;
    border-radius: 4px;
    color: #f1f1f2bf;
    margin: 8px 8px 0 0;

    &:not(:first-child)::before {
      width: 0;
    }

    &:hover,
    &:active {
      border: 1px solid #f1f1f2 !important;
      box-shadow: none;
    }

    &:not(.ant-radio-button-wrapper-disabled):first-child {
      border: 1px solid transparent;
    }
  }

  .ant-radio-button-wrapper-checked {
    color: #0f3f09;

    &:hover,
    &:active {
      color: #1d5811;
    }
  }

  .ant-table {
    background: #1c4f23;
  }

  .ant-table-body {
    overflow-x: scroll;
  }

  .ant-table-thead > tr > th {
    border-bottom: 1px solid #427136;
  }

  .ant-table-tbody > tr > td {
    padding: 24px 16px;
    border-bottom: 1px solid transparent;
  }

  .ant-table-tbody {
    &:not:first-child {
      border-bottom: 1px solid #387136;
    }
  }

  .ant-table-row:not(:last-child) {
    tr,
    td {
      border-bottom: 1px solid #36713b;
    }
  }

  .ant-table-row:hover {
    cursor: pointer;
  }

  .ant-table-placeholder {
    background: #2a4f1c;
    border-bottom: 0;

    .ant-empty {
      color: #f1f1f2bf;
    }
  }
}
</style>
