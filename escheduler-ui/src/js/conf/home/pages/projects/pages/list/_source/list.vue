<template>
  <div class="list-model">
    <div class="table-box">
      <table>
        <tr>
          <th>
            <span>{{$t('#')}}</span>
          </th>
          <th>
            <span>{{$t('Project Name')}}</span>
          </th>
          <th>
            <span>{{$t('Owned Users')}}</span>
          </th>
          <th>
            <span>{{$t('Process Define Count')}}</span>
          </th>
          <th>
            <span>{{$t('Process Instance Running Count')}}</span>
          </th>
          <th>
            <span>{{$t('Description')}}</span>
          </th>
          <th>
            <span>{{$t('Create Time')}}</span>
          </th>
          <th>
            <span>{{$t('Update Time')}}</span>
          </th>
          <th width="80">
            <span>{{$t('Operation')}}</span>
          </th>
        </tr>
        <tr v-for="(item, $index) in list" :key="$index">
          <td>
            <span>{{parseInt(pageNo === 1 ? ($index + 1) : (($index + 1) + (pageSize * (pageNo - 1))))}}</span>
          </td>
          <td>
            <span>
              <a href="javascript:" @click="_switchProjects(item)" class="links">{{item.name}}</a>
            </span>
          </td>
          <td>
            <span>{{item.userName || '-'}}</span>
          </td>
          <td>
            <span>{{item.defCount}}</span>
          </td>
          <td>
            <span>{{item.instRunningCount}}</span>
          </td>
          <td>
            <span>{{item.desc}}</span>
          </td>
          <td><span>{{item.createTime | formatDate}}</span></td>
          <td><span>{{item.updateTime | formatDate}}</span></td>

          <td>
            <x-button
                    type="info"
                    shape="circle"
                    size="xsmall"
                    data-toggle="tooltip"
                    :title="$t('Edit')"
                    @click="_edit(item)"
                    icon="iconfont icon-bianjixiugai"
                    v-ps="['GENERAL_USER']">
            </x-button>
            <x-poptip
                    :ref="'poptip-' + $index"
                    placement="bottom-end"
                    width="90">
              <p>{{$t('Delete?')}}</p>
              <div style="text-align: right; margin: 0;padding-top: 4px;">
                <x-button type="text" size="xsmall" shape="circle" @click="_closeDelete($index)">{{$t('Cancel')}}</x-button>
                <x-button type="primary" size="xsmall" shape="circle" @click="_delete(item,$index)">{{$t('Confirm')}}</x-button>
              </div>
              <template slot="reference">
                <x-button
                        type="error"
                        shape="circle"
                        size="xsmall"
                        data-toggle="tooltip"
                        :title="$t('delete')"
                        icon="iconfont icon-shanchu"
                        v-ps="['GENERAL_USER']">
                </x-button>
              </template>
            </x-poptip>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>
<script>
  import { mapActions, mapMutations } from 'vuex'
  import localStore from '@/module/util/localStorage'
  import { findComponentDownward } from '@/module/util/'

  export default {
    name: 'projects-list',
    data () {
      return {
        list: []
      }
    },
    props: {
      projectsList: Array,
      pageNo: Number,
      pageSize: Number
    },
    methods: {
      ...mapActions('projects', ['deleteProjects']),
      ...mapMutations('dag', ['setProjectName']),
      _switchProjects (item) {
        this.setProjectName(item.name)
        localStore.setItem('projectName', `${item.name}`)
        localStore.setItem('projectId', `${item.id}`)
        this.$router.push({ path: `/projects/index` })
      },
      _closeDelete (i) {
        this.$refs[`poptip-${i}`][0].doClose()
      },
      /**
       * Delete Project
       * @param item Current record
       * @param i index
       */
      _delete (item, i) {
        this.deleteProjects({
          projectId: item.id
        }).then(res => {
          this.$refs[`poptip-${i}`][0].doClose()
          this.list.splice(i, 1)
          this.$message.success(res.msg)
        }).catch(e => {
          this.$refs[`poptip-${i}`][0].doClose()
          this.$message.error(e.msg || '')
        })
      },
      /**
       * edit project
       * @param item Current record
       */
      _edit (item) {
        findComponentDownward(this.$root, 'projects-list')._create(item)
      }
    },
    watch: {
      projectsList (a) {
        this.list = []
        setTimeout(() => {
          this.list = a
        })
      }
    },
    created () {
    },
    mounted () {
      this.list = this.projectsList
    },
    components: { }
  }
</script>
