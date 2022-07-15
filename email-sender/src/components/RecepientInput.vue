<template lang="pug">
.email-wrapper
    .mailBox
      .mailBox__chips(v-for="(item, index) in selectedEmailsRaw")
        .chips
          .chips__name {{ item.fullName || '' }}
          .chips__email {{ item.email }}
          

      input(
        ref="input"
        type="text"
        v-model="searchValue"
        @input="showTooltip"
        @keyup.enter="addSelectedEmailByEnter"
        :placeholder="inputPlaceholder"
        @focusin="openOptions"
        :disabled="isDisabled"
      )

      .mailBox__clearAll(@click="removeAll" v-if="selectedEmailsRaw.length && isMultiOption" ) Clear All

    .selectBox(ref="option")
      .selectBox__container(v-if="isAllEmails && filteredList.length")
        .selectBox__settings(v-if="isMultiOption")
          .selectBox__settings-text(@click="selectAll") Select All

        .selectBox__option(v-for="(item, index) in filteredList" @click="addSelectedEmails(index)" )
          .selectBox__option-text
            .selectBox__option-text-name {{ item.fullName || '' }}
            .selectBox__option-text-email {{ item.email }}
            .chips__close
            .close(@click="deleteSelectedEmails(index)") X


</template>

<script>

export default {
  name: 'RecipientInput',
  props: {
    possibleEmails: {
      type: Array,
      default: () => []
      // ex. { _id, email, photo, fullName }
    },
    selectedEmail: {
      type: Array,
      default: () => []
      // ex. { _id, email, photo, fullName }
    },
    isMultiOption: {
      type: Boolean,
      default: true
    },
    isDisabled: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isAllEmails: false,
      searchValue: '',
      selectedEmailsRaw: JSON.parse(JSON.stringify(this.selectedEmail))
    }
  },
  computed: {
    notSelectedEmails() {
      return this.possibleEmails
          .filter(item => this.validEmail(item.email))
          .filter(item => !this.selectedEmailsRaw.some(i => item.email === i.email))
    },
    filteredList() {
      if (this.searchValue) return this.notSelectedEmails.filter(({ fullName, email }) => {
        const str = `${ fullName } ${ email }`.toLowerCase()
        return str.indexOf(this.searchValue.toLowerCase()) !== -1
      })
      return this.notSelectedEmails
    },
    inputPlaceholder() {
      if (this.isDisabled) return 'Option to Choose is disabled...'
      if (this.isMultiOption) return 'Choose or write an email address...'
      return 'Choose Single email address...'
    }
  },
  methods: {
    selectAll() {
      if (this.isDisabled) return
      this.selectedEmailsRaw = [ ...this.selectedEmailsRaw, ...this.notSelectedEmails ]
      this.emailActions()
    },
    removeAll() {
      if (this.isDisabled) return
      this.selectedEmailsRaw = []
      this.emailActions()
    },
    getContact(item) {
      return this.possibleEmails.find(i => i.email === item.email)
    },
    showTooltip() {
      if (this.validEmail(this.searchValue)) {
        this.alertToggle({ message: "Click enter to save", isShow: true, type: "success" })
      }
    },
    addSelectedEmailByEnter() {
      if (!this.isMultiOption) {
        this.clearInput()
        return
      }
      if (this.validEmail(this.searchValue)) if (!this.selectedEmailsRaw.some(i => i.email === this.searchValue)) {
        this.selectedEmailsRaw.push({ email: this.searchValue, fullName: this.searchValue })
        this.emailActions()
      }
      this.clearInput()
    },
    addSelectedEmails(index) {
      const selectedElem = this.filteredList[index]
      if (!this.isMultiOption) {
        if (!this.selectedEmailsRaw.length) {
          this.selectedEmailsRaw.push(selectedElem)
          this.emailActions()
          this.clearInput()
          return
        }
        this.selectedEmailsRaw = []
        this.selectedEmailsRaw.push(selectedElem)
        this.emailActions()
        this.clearInput()
        return
      }
      this.selectedEmailsRaw.push(selectedElem)
      this.emailActions()
      this.clearInput()
    },
    deleteSelectedEmails(index) {
      if (this.isDisabled) return
      this.selectedEmailsRaw.splice(index, 1)
      this.emailActions()
    },
    clearInput() {
      this.searchValue = ''
    },
    closeOptions() {
      this.isAllEmails = false
    },
    openOptions() {
      if (this.isDisabled) return
      this.isAllEmails = true
    },
    validEmail(email) {
      const re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      return re.test(email)
    },
    optionsClickOutside(e) {
      if (this.notSelectedEmails.length) if (!this.$refs.option.contains(e.target) && this.$refs.input !== document.activeElement && e.target.className !== 'close') this.closeOptions()
    },
    emailActions() {
      this.$emit('emailAction', this.selectedEmailsRaw)
    }
  },
  mounted() {
    document.addEventListener('click', this.optionsClickOutside)
  },
  unmounted() {
    document.removeEventListener('click', this.optionsClickOutside, false)
  }
}
</script>

<style lang="scss" scoped>

.email-wrapper {
  position: relative;
  width: 100%;
}

.selectBox {
  z-index: 5555;
  position: absolute;
  width: 100%;

  &__container {
    background: white;
    max-height: 380px;
    overflow: auto;
    border: 1px solid lightgray;
    border-top: none;
  }

  &__settings {
    padding: 11px 20px 9px;

    &-text {
      color: green;
      cursor: pointer;
      display: flex;
      justify-content: start;

      &:hover {
        text-decoration: underline;
      }
    }
  }

  &__option {
    display: flex;
    padding: 8px 20px;
    cursor: pointer;
    transition: .1s ease-out;
    align-items: center;
    gap: 20px;

    &:hover {
      background: lightgray;
    }

    &-text {
      &-name {
        font-family: 'Myriad600';
        margin-bottom: 4px;
      }

      &-email {
        color: gray;
      }
    }
  }
}

.mailBox {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-content: flex-start;
  align-items: flex-start;
  position: relative;
  gap: 6px 10px;
  border: 1px solid gary;
  border-radius: 2px;
  padding: 6px 70px 6px 6px;

  &__clearAll {
    position: absolute;
    background: white;
    right: 10px;
    top: 15px;
    color: green;
    cursor: pointer;
    display: flex;
    justify-content: start;

    &:hover {
      text-decoration: underline;
    }
  }

  input {
    width: 220px;
    outline: none;
    border: none;
    font-size: 14px;
    height: 32px;
    padding: 0 7px;
  }

  input[type=text]:disabled {
    background: white;
  }

  &__chips {
    display: flex;
  }
}

.chips {
  display: flex;
  align-items: center;
  padding: 0 8px;
  gap: 6px;
  background: lightblue;
  border-radius: 2px;
  height: 32px;

  &__name {
    font-family: 'Myriad600';
    letter-spacing: -0.1px;
  }

  &__email {
    color: gray;
  }

  &__close {
    position: relative;
    width: 19px;
    margin-top: -17px;
    transform: scale(0.8);
  }
}
</style>
