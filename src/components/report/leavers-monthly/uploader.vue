<template>
  <q-dialog
    ref="dialog"
    @hide="onDialogHide"
  >
    <q-card
      class="q-dialog-plugin overflow-hidden"
      style="width: 35%;"
    >
      <q-card-section>
        <div class="text-weight-medium text-caption text-justify text-deep-orange q-mb-sm">
          *Be advised that the uploader is <span class="text-weight-bold">case</span>, <span class="text-weight-bold">spelling</span> and <span class="text-weight-bold">column</span> sensitive.
          You may refer to the export file for reference.
        </div>
        <q-file
          outlined
          counter
          accept=".xlsx"
          :label="`Upload data for ${brand} Leavers Monthly (.xlsx)`"
          v-model="file"
          :disable="loading"
          :loading="loading"
          @input="processFile"
        >
          <template v-slot:prepend>
            <q-icon name="attach_file" />
          </template>
        </q-file>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { positive, negative } from 'boot/notifier'
import PostRepo from 'src/repository/post'

export default {
  props: ['brand'],

  data () {
    return {
      file: null,
      loading: false
    }
  },

  methods: {
    async processFile (file) {
      this.loading = true
      const { domain } = this.$q.localStorage.getItem('userData')

      try {
        const formData = new FormData()
        formData.append('file', file)

        const result = await PostRepo[`UploadLeaverMonthly${this.brand}File`](domain, formData)
        console.log(result)

        this.loading = true
        positive('Success', 'data has been uploaded')

        setTimeout(() => {
          this.$router.go()
        }, 1200)
      } catch (err) {
        this.loading = true

        console.log(err)
        negative('Error encountered', 'data was not uploaded')
      }
    },

    show () {
      this.$refs.dialog.show()
    },
    hide () {
      this.$refs.dialog.hide()
    },
    onDialogHide () {
      this.$emit('hide')
    },
    onOKClick () {
      this.$emit('ok')
      this.hide()
    },
    onCancelClick () {
      this.hide()
    }
  }
}
</script>
