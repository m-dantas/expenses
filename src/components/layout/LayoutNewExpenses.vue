<template>
    <div class="w-100 d-flex justify-content-center">

      <button @click="showModal = true" class="btn btn-lg btn-outline-primary w-75">
        <i class="fa fa-plus"></i>
        Novo Gasto
      </button>

      <form @submit.prevent="submit()">
        <div class="modal fade" :class="{show: showModal}" style="display: block;" :style="{display: showModal ? 'block' : 'none'}">
          <div class="modal-dialog modal-lg" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="exampleModalLiveLabel">Adicionar um novo gasto</h5>
                <button type="button" @click="closeModal()" class="close">
                  <span aria-hidden="true">×</span>
                </button>
              </div>
              <div class="modal-body">
                <div class="row">
                  <div class="form-group col-8">
                    <label for="">Descrição:</label>
                    <input
                      type="text"
                      class="form-control"
                      v-model="form.description"
                      required
                    >
                  </div>
                  <div class="form-group col-4">
                    <label for="">Valor (R$):</label>
                    <input
                      type="text"
                      class="form-control"
                      v-model="form.value"
                      required
                    />
                  </div>
                </div>
                <div class="form-group col-12 flex-column d-flex align-items-center">
                    <input
                      ref="input"
                      type="file"
                      class="form-control d-none"
                      accept="image/*"
                      @change="handleFile($event)"
                    >
                    <button
                      type="button"
                      @click="openFileDialog()"
                      class="btn w-50 btn-outline-secondary"
                    >
                      Adicionar comprovante
                    </button>

                    <div class="mt-2" v-if="form.receipt">
                      {{form.receipt.name}}
                      <button
                        type="button"
                        @click="form.receipt = ''"
                        class="btn badge badge-light"
                      >
                        <i class="fa fa-trash text-danger"></i>
                      </button>
                    </div>
                </div>
              </div>
              <div class="modal-footer">
                <button type="button" @click="closeModal()" class="btn btn-secondary">Fechar</button>
                <button class="btn btn-primary" :disabled="loading">
                  <template v-if="loading">
                    <i class="fa fa-spin fa-spinner"></i>
                    Incluindo...
                  </template>
                  <template v-else>
                    Incluir novo gasto
                  </template>
                </button>
              </div>
            </div>
          </div>
        </div>
      </form>

      <div class="modal-backdrop fade" :style="{display: showModal ? 'block' : 'none'}" :class="{show: showModal}"></div>

    </div>

</template>

<script>
export default {
  data: () => ({
    loading: false,
    showModal: false,
    form: {
      description: "",
      value: "",
      receipt:""
    }
  }),
  computed: {
    fileName (){
      const { receipt } = this.form

      if(receipt){
        const split = receipt.name.split('.')
        return `${split[0]}-${new Date().getTime()}.${split[1]}`
      } else {
        return ''
      }
    }
  },
  methods: {
    openFileDialog (){
      this.$refs.input.value = null
      this.$refs.input.click()
    },
    handleFile ({ target }){
      this.form.receipt = target.files[0]
    },
    async submit (){
      let url = ''
      this.loading = true

      try {
        this.$root.$emit('Spinner::show')
        const ref = this.$firebase.database().ref(window.uid)
        const id = ref.push().key

        if(this.form.receipt){
          const snapshot = await this.$firebase.storage()
          .ref(window.uid)
          .child(this.fileName)
          .put(this.form.receipt)

          url = await snapshot.ref.getDownloadURL()

        }
        const payload = {
          id,
          ... this.form,
          receipt: url,
          createdAt: new Date().getTime(),
        }

        ref.child(id).set(payload, err=>{
          this.$root.$emit('Spinner::hide')

          if(err){
            this.$root.$emit('Notification::show', {
              type: 'danger',
              message: 'Não foi possivel inserir o gasto, tente novamente!'
            })
            this.closeModal()
            this.loading = false
          }else{
            this.$root.$emit('Notification::show', {
              type: 'success',
              message: 'Gasto inserido com sucesso'
            })
            this.closeModal()
            this.loading = false            
          }
        });
      } catch (error) {
        this.$root.$emit('Notification::show', {
          type: 'danger',
          message: 'Não foi possivel inserir o gasto, tente novamente!'
        })
      } finally {
        this.$root.$emit('Spinner::hide')
        this.loading = false
      }
    },
    closeModal () {
      this.showModal = false
    }
  }
}
</script>

<style lang="scss" scoped>
.modal {
  color: var(--darker);
}
</style>
