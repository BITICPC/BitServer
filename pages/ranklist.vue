<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="12" md="10" lg="8" xl="6">
        <v-card>
          <!-- <v-card-title>
            用户排名
          </v-card-title>
          <v-card-subtitle>
            User Ranklist
          </v-card-subtitle> -->
          <v-data-table
            :headers="headers"
            :items="ranklist"
            :options.sync="options"
            :search="search"
            :loading="loading"
            loading-text="正在加载数据，请稍等..."
            disable-filtering
            hide-default-footer
          >
            <!-- <template v-slot:top>
              <v-toolbar flat>
                <v-toolbar-title>用户排名</v-toolbar-title>
                <v-divider class="mx-4" inset vertical />
                <v-spacer />
                <v-text-field
                  v-model="search"
                  color="purple"
                  label="请输入关键字"
                  append-icon="mdi-magnify"
                  single-line
                  hide-details
                  @click:append="getRanklist(search)"
                />
              </v-toolbar>
            </template> -->
            <template v-slot:item="user">
              <tr :align="user.headers[0].align" class="d-none d-sm-table-row">
                <td>{{ page.itemsPerPage * (page.index - 1) + user.index + 1 }}</td>
                <td>
                  <nuxt-link :to="`/user/${user.item.username}`">
                    {{ user.item.username }}
                  </nuxt-link>
                </td>
                <td>{{ user.item.signature }}</td>
                <td>{{ user.item.accepted }}</td>
                <td>{{ user.item.attempted }}</td>
              </tr>
              <tr class="hidden-sm-and-up">
                <td class="v-data-table__mobile-row">
                  <div class="v-data-table__mobile-row__header">
                    {{ user.headers[0].text }}
                  </div>
                  <div class="v-data-table__mobile-row__cell">
                    {{ page.itemsPerPage * (page.index - 1) + user.index + 1 }}
                  </div>
                </td>
                <td class="v-data-table__mobile-row">
                  <div class="v-data-table__mobile-row__header">
                    {{ user.headers[1].text }}
                  </div>
                  <div class="v-data-table__mobile-row__cell">
                    <nuxt-link :to="`/user/${user.item.username}`">
                      {{ user.item.username }}
                    </nuxt-link>
                  </div>
                </td>
                <td class="v-data-table__mobile-row">
                  <div class="v-data-table__mobile-row__header">
                    {{ user.headers[2].text }}
                  </div>
                  <div class="v-data-table__mobile-row__cell">
                    {{ user.item.signature }}
                  </div>
                </td>
                <td class="v-data-table__mobile-row">
                  <div class="v-data-table__mobile-row__header">
                    {{ user.headers[3].text }}
                  </div>
                  <div class="v-data-table__mobile-row__cell">
                    {{ user.item.accepted }}
                  </div>
                </td>
                <td class="v-data-table__mobile-row">
                  <div class="v-data-table__mobile-row__header">
                    {{ user.headers[4].text }}
                  </div>
                  <div class="v-data-table__mobile-row__cell">
                    {{ user.item.attempted }}
                  </div>
                </td>
              </tr>
            </template>
          </v-data-table>
        </v-card>
        <div class="text-center pt-2">
          <v-pagination v-model="page.index" :length="page.count" color="purple" />
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import api from '@/plugins/utils/api'

export default {
  data () {
    return {
      ranklist: [],
      headers: [
        {
          text: '排名',
          align: 'center',
          sortable: false,
          filterable: false,
          value: 'index'
        },
        {
          text: '用户名',
          align: 'center',
          sortable: true,
          filterable: true,
          value: 'username'
        },
        {
          text: '个性签名',
          align: 'center',
          sortable: false,
          filterable: false,
          value: 'signature'
        },
        {
          text: '通过数量',
          width: '120',
          align: 'center',
          sortable: true,
          filterable: false,
          value: 'accepted'
        },
        {
          text: '尝试数量',
          width: '120',
          align: 'center',
          sortable: true,
          filterable: false,
          value: 'attempted'
        }
      ],
      options: {},
      page: {
        index: 1,
        count: -1,
        itemsPerPage: 10
      },
      search: '',
      loading: false
    }
  },
  watch: {
    'page.index' () {
      this.getRanklist()
    },
    options: {
      handler () {
        this.getRanklist()
      }
    }
  },
  methods: {
    getSortBy () {
      if (this.options.sortBy.length > 0 && this.options.sortBy[0] === 'attempted') {
        return 'TotalProblemsAttempted'
      }
      return 'TotalProblemsAccepted'
    },
    getSortDesc () {
      return this.options.sortDesc.length > 0 ? this.options.sortDesc[0] : true
    },
    getRanklist (username) {
      this.loading = true
      this.ranklist = []
      api.getRanklist({
        by: this.getSortBy(),
        descend: this.getSortDesc(),
        page: this.page.index - 1,
        itemsPerPage: this.page.itemsPerPage
      }).then((res) => {
        res.data.forEach((user, index) => {
          if (username === undefined || user.username.includes(username)) {
            this.ranklist.push({
              username: user.username,
              signature: '这个家伙很懒，什么都没有写',
              accepted: user.totalProblemsAccepted,
              attempted: user.totalProblemsAttempted
            })
          }
        })
        if (this.page.count === -1) {
          this.page.count = Math.ceil(res.headers['x-bitwaves-count'] / this.page.itemsPerPage)
        }
      }).finally(() => { this.loading = false })
    }
  }
}
</script>
