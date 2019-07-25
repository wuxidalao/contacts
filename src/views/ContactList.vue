<template>
  <div>
    <!-- 联系人卡片 -->
    <van-contact-card
      :type="cardType"
      :name="currentContact.name"
      :tel="currentContact.tel"
      @click="showList = true"
    />

    <!-- 联系人列表 -->
    <van-popup v-model="showList" position="bottom">
      <van-contact-list
        v-model="chosenContactId"
        :list="list"
        @add="onAdd"
        @edit="onEdit"
        @select="onSelect"
      />
    </van-popup>

    <!-- 联系人编辑 -->
    <van-popup v-model="showEdit" position="bottom">
      <van-contact-edit
        :contact-info="editingContact"
        :is-edit="isEdit"
        @save="onSave"
        @delete="onDelete"
      />
    </van-popup>
  </div>
</template>

<script>
import axios from 'axios'
import { ContactCard, ContactList, ContactEdit, popup, Toast } from 'vant'
export default {
  name: 'ContactList',
  data () {
    return {
      chosenContactId: null,
      editingContact: {},
      showList: false,
      showEdit: false,
      isEdit: false,
      list: [],
      instance: null
    }
  },

  created () {
    this.instance = axios.create({
      baseUrl: 'http://localhost:9000/api',
      timeout: 1000
    })

    this.instance.get('/contactList')
      .then(res => {
        console.log(res)
        this.list = res.data.data
      }).catch(err => {
        if (err) {
          Toast('请求错误' + err.Code)
        }
      })
  },

  components: {
    [ContactCard.name]: ContactCard,
    [ContactList.name]: ContactList,
    [ContactEdit.name]: ContactEdit,
    [popup.name]: popup
  },

  computed: {
    cardType () {
      return this.chosenContactId !== null ? 'edit' : 'add'
    },

    currentContact () {
      const id = this.chosenContactId
      return id !== null ? this.list.filter(item => item.id === id)[0] : {}
    }
  },

  methods: {
    // 添加联系人
    onAdd () {
      this.editingContact = { id: this.list.length }
      this.isEdit = false
      this.showEdit = true
    },

    // 编辑联系人
    onEdit (item) {
      this.isEdit = true
      this.showEdit = true
      this.editingContact = item
    },

    // 选中联系人
    onSelect () {
      this.showList = false
    },

    // 保存联系人
    onSave (info) {
      this.instance.post('/contact/new/json', info)
        .then(res => {
          this.showEdit = false
          this.showList = false
          if (this.isEdit) {
            this.list = this.list.map(item => item.id === info.id ? info : item)
          } else {
            this.list.push(info)
          }
          this.chosenContactId = info.id
        }).catch(err => {
          if (err) {
            Toast('请求错误')
          }
        })
    },

    // 删除联系人
    onDelete (info) {
      this.instance.delete('/contact', {
        params: {
          id: info.id
        }
      }).then(res => {
        this.showEdit = false
        this.list = this.list.filter(item => item.id !== info.id)
        if (this.chosenContactId === info.id) {
          this.chosenContactId = null
        }
      }).catch(err => {
        if (err) {
          Toast('请求错误')
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
