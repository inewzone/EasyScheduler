<template>
  <m-popup
          ref="popup"
          :ok-text="item ? $t('Edit') : $t('Submit')"
          :nameText="item ? $t('Edit Tenant') : $t('Create Tenant')"
          @ok="_ok">
    <template slot="content">
      <div class="create-tenement-model">
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('Tenant Code')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    :disabled="item ? true : false"
                    v-model="tenantCode"
                    :placeholder="$t('Please enter name')">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('Tenant Name')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="tenantName"
                    :placeholder="$t('Please enter name')"
                    autocomplete="off">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('Queue')}}</template>
          <template slot="content">
            <x-select v-model="queueId">
              <x-option
                      v-for="city in queueList"
                      :key="city.id"
                      :value="city"
                      :label="city.code">
              </x-option>
            </x-select>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name">{{$t('Description')}}</template>
          <template slot="content">
            <x-input
                    type="textarea"
                    v-model="desc"
                    :placeholder="$t('Please enter description')"
                    autocomplete="off">
            </x-input>
          </template>
        </m-list-box-f>
      </div>
    </template>
  </m-popup>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import store from '@/conf/home/store'
  import mPopup from '@/module/components/popup/popup'
  import mListBoxF from '@/module/components/listBoxF/listBoxF'

  export default {
    name: 'create-tenement',
    data () {
      return {
        store,
        queueList: [],
        queueId: {},
        tenantCode: '',
        tenantName: '',
        desc: ''
      }
    },
    props: {
      item: Object
    },
    methods: {
      _ok () {
        if (this._verification()) {
          // The name is not verified
          if (this.item && this.item.groupName === this.groupName) {
            this._submit()
            return
          }
          // Verify username
          this.store.dispatch(`security/verifyName`, {
            type: 'tenant',
            tenantCode: this.tenantCode
          }).then(res => {
            this._submit()
          }).catch(e => {
            this.$message.error(e.msg || '')
          })
        }
      },
      _getQueueList () {
        return new Promise((resolve, reject) => {
          this.store.dispatch('security/getQueueList').then(res => {
            this.queueList = _.map(res, v => {
              return {
                id: v.id,
                code: v.queueName
              }
            })
            this.$nextTick(() => {
              this.queueId = this.queueList[0]
            })
            resolve()
          })
        })
      },
      _verification () {
        let isEn = /^[0-9a-zA-Z_.-]{1,}$/

        if (!this.tenantCode) {
          this.$message.warning(`${i18n.$t('Please enter the tenant code in English')}`)
          return false
        }
        if (!isEn.test(this.tenantCode) || _.startsWith(this.tenantCode, '_', 0) || _.startsWith(this.tenantCode, '.', 0)) {
          this.$message.warning(`${i18n.$t('Please enter tenant code in English')}`)
          return false
        }
        if (!this.tenantName) {
          this.$message.warning(`${i18n.$t('Please enter name')}`)
          return false
        }
        return true
      },
      _submit () {
        // 提交
        let param = {
          tenantCode: this.tenantCode,
          tenantName: this.tenantName,
          queueId: this.queueId.id,
          desc: this.desc
        }
        if (this.item) {
          param.id = this.item.id
        }

        this.$refs['popup'].spinnerLoading = true
        this.store.dispatch(`security/${this.item ? 'updateQueue' : 'createQueue'}`, param).then(res => {
          this.$emit('onUpdate')
          this.$message.success(res.msg)
          setTimeout(() => {
            this.$refs['popup'].spinnerLoading = false
          }, 800)
        }).catch(e => {
          this.$message.error(e.msg || '')
          this.$refs['popup'].spinnerLoading = false
        })
      }
    },
    watch: {
    },
    created () {
      this._getQueueList().then(res => {
        if (this.item) {
          this.$nextTick(() => {
            this.queueId = _.find(this.queueList, ['id', this.item.queueId])
          })
          this.tenantCode = this.item.tenantCode
          this.tenantName = this.item.tenantName
          this.desc = this.item.desc
        }
      })
    },
    mounted () {

    },
    components: { mPopup, mListBoxF }
  }
</script>