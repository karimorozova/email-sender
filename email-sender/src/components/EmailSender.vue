<template lang="pug">
.mail-wrapper
    .mail-wrapper__close(@click="closeEmailSender") X
    .mail-wrapper__title {{ title }}

    .mail-wrapper__input
      .mail-wrapper__label From:
      .mail-wrapper__wrap
        RecepientInput(
          :isMultiOption="false"
          :selectedEmail="from"
          :possibleEmails="possibleFromEmails"
          :isDisabled="isDisableSendFrom"
          @emailAction="(arr) => fillEmails('from', arr)"
        )

    .mail-wrapper__input
      .mail-wrapper__label Send To:
      .mail-wrapper__wrap
        RecepientInput(
          :selectedEmail="to"
          :possibleEmails="possibleToEmails"
          @emailAction="(arr) => fillEmails('to', arr)"
        )

    .mail-wrapper__input
      .mail-wrapper__label CC:
      .mail-wrapper__wrap
        RecepientInput(
          :selectedEmail="cc"
          :possibleEmails="possibleCCEmails"
          @emailAction="(arr) => fillEmails('cc', arr)"
        )
    .mail-wrapper__input
      .mail-wrapper__label
      .mail-wrapper__BccOption
        span(@click="toggleBcc") {{ isBccActiveRow || bcc.length ? 'Clear BCC' : 'Add BCC' }}

    .mail-wrapper__input(v-if="isBccActiveRow || bcc.length")
      .mail-wrapper__label BCC:
      .mail-wrapper__wrap
        RecepientInput(
          :selectedEmail="bcc"
          :possibleEmails="possibleBCCEmails"
          @emailAction="(arr) => fillEmails('bcc', arr)"
        )

    .mail-wrapper__body
      .mail-wrapper__label Subject:
      .mail-wrapper__subject
        input(type="text" v-model="subject" )
    .mail-wrapper__body
      .mail-wrapper__label.mail-wrapper__label-body Body:
    //-   .mail-wrapper__subject
    //-     ckeditor(v-model="html" :config="editorConfig")


    // SOON
    //.mail-wrapper__files(v-if="defaultAttachments.length" )
    //  .files-wrap(v-for="(item, index) in defaultAttachments" )
    //    .files-wrap--wrap
    //      CheckBox(:isChecked="item.isSent" @uncheck="deleteAttachment(item, index)" @check="addAttachment(item, index)")
    //      .files-wrap--label Attachment:
    //    .files-wrap__name {{item.fileName}}
    //input.mail-wrapper__upload(type="file" @change='uploadFile' :multiple='true')

    .mail-wrapper__buttons
      button(@click="sendEmail") Send
      button(@click="closeEmailSender") Close


</template>

<script>
import RecepientInput from "./RecepientInput"

export default {
  name: 'EmailSender',
  components: { RecepientInput, },
  props: {
    title: {
      type: String,
      default: 'Send Email'
    },
    selectedFrom: {
      type: Array,
      default: () => []
    },
    selectedTo: {
      type: Array,
      default: () => []
    },
    possibleToEmails: {
      type: Array,
      default: () => []
    },
    selectedCC: {
      type: Array,
      default: () => []
    },
    possibleCCEmails: {
      type: Array,
      default: () => []
    },
    selectedBCC: {
      type: Array,
      default: () => []
    },
    possibleBCCEmails: {
      type: Array,
      default: () => []
    },
    subjectRaw: {
      type: String,
      default: 'Message'
    },
    htmlRaw: {
      type: String,
      default: ''
    },
    isDisableSendFrom: {
      type: Boolean,
      default: false
    }
    // files: {
    //   type: Array,
    //   default: () => []
    // }
  },
  data() {
    return {
      from: [ ...this.selectedFrom ],
      to: [ ...this.selectedTo ],
      cc: [ ...this.selectedCC ],
      bcc: [ ...this.selectedBCC ],
      subject: JSON.parse(JSON.stringify(this.subjectRaw)),
      html: JSON.parse(JSON.stringify(this.htmlRaw)),
      users: [],
      user: {},
      vendors: [],
      clients: [],



      defaultAttachments: [],
      rawAttachments: [],
      isBccActiveRow: false,
      // isChecked: true,

      editorConfig: {
        allowedContent: true,
        uiColor: "#ffffff",
        height: 500
      }
    }
  },
  computed: {
    possibleFromEmails() {
      return this.users
          .filter(i => i.isActive)
          .map(({ _id, email, photo, firstName, lastName }) => ({ _id, email, photo, fullName: `${ firstName } ${ lastName }` }))
    }
  },
  methods: {
    fillEmails(prop, val) {
      this[prop] = val
    },
    toggleBcc() {
      if (this.isBccActiveRow) this.bcc = []
      this.isBccActiveRow = !this.isBccActiveRow
    },

    // uploadFile(e) {
    //   console.log(e.target.files)
    //   this.fileData = e.target.files
    // },
    // deleteAttachment(item, i) {
    //   item.isSent = false
    // },
    // addAttachment(item, i) {
    //   item.isSent = true
    // },
    async sendEmail() {
      if (!this.from.length) return
      if (!this.to.length) return
      const emailOnly = i => i.email

      this.$emit('sendEmail', {
        from: this.from[0].email,
        sender: this.from[0].fullName,
        to: this.to.map(emailOnly),
        cc: this.cc.map(emailOnly),
        bcc: this.bcc.map(emailOnly),
        subject: this.subject,
        html: this.html
        // rawAttachments: this.fileData,
        // defaultAttachments: this.defaultAttachments.filter(({ isSent }) => isSent === true)
      })
    },
    closeEmailSender() {
      this.$emit("close")
    }
  },
  mounted() {
    let [ elem ] = document.getElementsByTagName('body')
    elem.classList.add("hiddenScroll")
  },
  unmounted() {
    let [ elem ] = document.getElementsByTagName('body')
    elem.classList.remove("hiddenScroll")
  }
}
</script>

<style lang="scss" scoped>

.mail-wrapper {
  position: fixed;
  left: 255px;
  top: 0;
  box-sizing: border-box;
  width: calc(100% - 256px);
  padding: 50px;
  background: white;
  z-index: 30000;
  height: 100vh;
  overflow: auto;

  &__title {
    margin-bottom: 25px;
    padding-bottom: 12px;
    font-size: 16px;
    font-family: Myriad600;
    border-bottom: 1px solid lightgray;
  }

  &__close {
    top: 50px;
    right: 50px;
  }

  &__input {
    display: flex;
    margin-bottom: 15px;
  }

  &__label {
    width: 95px;
    margin-top: 16px;

    &-body {
      margin-top: 98px;
    }
  }

  &__wrap {
    width: calc(100% - 95px);
  }

  &__BccOption {
    color: green;
    cursor: pointer;
    display: flex;
    justify-content: start;

    &:hover {
      text-decoration: underline;
    }
  }

  &__body {
    display: flex;
  }

  &__subject {
    width: calc(100% - 95px);

    input {
      width: 100%;
      outline: none;
      height: 46px;
      border: 1px solid gray;
      padding: 0 20px;
      box-sizing: border-box;
      border-bottom: none;
    }
  }

  &__buttons {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 20px;
    margin-top: 50px;
  }
 
}
</style>
