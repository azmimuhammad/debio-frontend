<style lang="scss" scoped>
.on-hover {
  cursor: pointer;
}
.image-placeholder{
  cursor: pointer;
  border: 1px solid lightgrey;
}
.services__modal-title {
  word-break: break-word;
}
</style>

<template>
  <div>
    <DialogErrorBalance
      :show="isShowError"
      @close="closeDialog"
    />
    <v-container>
      <v-dialog v-if="messageWarning" v-model="showModalAlert" persistent width="450">
        <v-card>
          <v-card-title class="services__modal-title">{{ messageWarning.title }}</v-card-title>
          <v-card-subtitle class="mt-1" v-html="messageWarning.subtitle"></v-card-subtitle>
          <v-card-actions>
            <v-btn
              block color="primary"
              @click="handleRedirect"
            >
              {{ messageWarning.actionTitle }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-row>
        <v-col cols="12" xl="8" lg="8" md="8" order-md="1" order="2">
            <v-card class="dg-card" elevation="0" outlined>
              <v-form ref="addServiceForm">
                <v-card-text class="px-8 pb-8 pt-10">
                  <div class="mt-5 mb-12 justify-space-evenly" align="center">
                      <v-avatar
                        size="125"
                        @click="selectPicture"
                        rounded
                        class="image-placeholder"
                      >
                        <v-img v-if="!imageUrl" src="@/assets/add-image-placeholder.png" alt="image"></v-img>
                        <v-img v-else :src="imageUrl" alt="image"></v-img>
                        <v-file-input
                          style="display: none"
                          hide-input
                          prepend-icon="mdi-camera"
                          accept="image/*"
                          ref="fileInput"
                          :disabled="isLoading"
                          @change="imageUploadEventListener"
                        />
                      </v-avatar>

                  </div>
                    <v-select
                      dense
                      label="Service Category"
                      placeholder="Service Category"
                      v-model="document.category"
                      outlined
                      :items="listCategories"
                      :disabled="hasServicePayload || isLoading"
                      item-text="service_categories"
                      item-value="service_categories"
                      :rules="fieldRequiredRule"
                    />

                    <v-select
                      dense
                      label="Type of Biological Sample"
                      placeholder="Type of Biological Sample"
                      v-model="document.dnaCollectionProcess"
                      outlined
                      :items="dnaCollectionProcessList"
                      item-text="name"
                      item-value="collectionProcess"
                      :disabled="isLoading"
                      :rules="fieldRequiredRule"
                    />

                    <v-text-field
                      dense
                      label="Service Name"
                      placeholder="Service Name"
                      outlined
                      v-model="document.name"
                      :disabled="isLoading"
                      :rules="[...fieldRequiredRule, ...serviceNameRules, ...fieldEnglishRules]"
                    />

                    <div class="d-flex">
                      <v-row>
                        <v-col>
                          <v-select
                            label="Currency"
                            outlined
                            dense
                            max="30"
                            v-model="document.currency"
                            :items="currencyList"
                            :rules="fieldRequiredRule"
                            :disabled="hasServicePayload || isLoading"
                          />
                        </v-col>
                        <v-col>
                          <v-text-field
                            dense
                            label="Price"
                            placeholder="e.g. 20.005"
                            outlined
                            v-model.number="document.price"
                            type="number"
                            min="0"
                            step=".001"
                            :disabled="isLoading"
                            :hint="priceHint"
                            :rules="[...fieldRequiredRule, ...decimalRule]"
                          />
                        </v-col>
                        <v-col>
                          <v-select
                            :disabled="isBiomedical || hasServicePayload ||isLoading"
                            label="QC Currency"
                            outlined
                            dense
                            v-model="document.currency"
                            :items="currencyList"
                            :rules="fieldRequiredRule"
                          />
                        </v-col>
                        <v-col>
                          <v-text-field
                            :disabled="isBiomedical || isLoading"
                            dense
                            label="QC Price"
                            placeholder="e.g. 20.005"
                            outlined
                            v-model.number="document.qcPrice"
                            type="number"
                            min="0"
                            step=".001"
                            :hint="qcPriceHint"
                            :rules="[...fieldRequiredRule, ...decimalRule]"
                          />
                        </v-col>
                      </v-row>
                    </div>

                    <div class="grey--text mb-6">
                      *Every transaction fee includes a 5% service charge.
                    </div>

                    <v-text-field
                      dense
                      label="Short Description"
                      placeholder="Short Description"
                      outlined
                      v-model="document.description"
                      :disabled="isLoading"
                      :rules="[...fieldRequiredRule, ...descriptionRules, ...fieldEnglishRules]"
                    />

                    <v-row >
                      <v-col cols="8">
                        <v-text-field
                          dense
                          label="Expected Duration"
                          placeholder="Expected Duration"
                          min="0"
                          max="30"
                          outlined
                          type="number"
                          v-model.number="document.duration"
                          :disabled="isLoading"
                          :rules="fieldRequiredRule"
                        />
                      </v-col>
                      <v-col cols="4" style="position: relative">
                        <v-select
                          label="Duration Type"
                          outlined
                          dense
                          v-model="document.durationType"
                          :disabled="isLoading"
                          :items="listExpectedDuration"
                          :rules="fieldRequiredRule"
                        />
                      </v-col>
                    </v-row>


                    <v-textarea
                      label="Long Description"
                      placeholder="Long Description"
                      outlined
                      v-model="document.longDescription"
                      :disabled="isLoading"
                      :rules="[...fieldRequiredRule, ...longDescriptionRules]"
                    />

                    <v-file-input
                      :disabled="isLoading"
                      :rules="fileInputRules"
                      accept=".pdf"
                      dense
                      label="Test Result Sample"
                      placeholder="Test Result Sample"
                      prepend-icon="mdi-file-document"
                      outlined
                      v-model="testResultSampleFile"
                      @change="fileUploadEventListener"
                    />

                    <div class= "d-flex justify-space-between" >
                      <div class="mb-5">
                        <span
                          style="font-size: 12px"
                        > Estimated Transaction Weight </span>
                          <v-tooltip bottom>
                            <template v-slot:activator="{ on, attrs }">
                              <v-icon
                                color="primary"
                                size="14"
                                v-bind="attrs"
                                v-on="on"
                              > mdi-alert-circle-outline
                              </v-icon>
                            </template>
                            <span style="font-size: 10px;">Total fee paid in DBIO to execute this transaction.</span>
                          </v-tooltip>
                      </div>
                      <span style="font-size: 12px;">
                        {{ fee }}
                      </span>
                    </div>

                    <v-btn
                      color="primary"
                      block
                      large
                      :disabled="isLoading"
                      :loading="isLoading"
                      @click="handleCreateService"
                    >
                      Submit
                    </v-btn>
                </v-card-text>
              </v-form>
            </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapState } from "vuex"
import { uploadFile, getFileUrl } from "@/lib/pinata-proxy"
import { createService, createServiceFee, claimRequestService } from "@/lib/polkadotProvider/command/services"
import { queryLabsById } from "@/lib/polkadotProvider/query/labs";
import { getProvideRequestService, getCategories, getConversionCache, getDNACollectionProcess } from "@/lib/api";
import { toEther } from "@/lib/balance-format"
import { generalDebounce } from "@/utils"
import DialogErrorBalance from "@/components/Dialog/DialogErrorBalance"

const englishAlphabet = val => (val && /^[A-Za-z0-9!@#$%^&*\\(\\)\-_=+:;"',.\\/? ]+$/.test(val)) || "This field can only contain English alphabet"

export default {
  name: "AddLabServices",
  components: { DialogErrorBalance },
  data: () => ({
    document: {
      category: "",
      dnaCollectionProcess: "",
      name: "",
      currency: "DAI",
      price: 0,
      qcPrice: 0,
      description: "",
      longDescription: "",
      duration: "",
      durationType: "Days"
    },
    imageUrl: "",
    testResultSampleUrl: "",
    currentDAIprice: 0,
    statusLab: null,
    labInfo: null,
    messageWarning: {},
    serviceFlow: "RequestTest",
    files: [],
    testResultSampleFile:[],
    listCategories:[],
    isLoading: false,
    showModalAlert: false,
    currencyList: ["DAI"],
    listExpectedDuration: ["WorkingDays", "Hours", "Days"],
    dnaCollectionProcessList: [],
    isBiomedical: false,
    isShowError: false,
    fee: 0
  }),

  computed: {
    ...mapState({
      servicePayload: (state) => state.lab.providePayload,
      api: (state) => state.substrate.api,
      exist: (state) => state.substrate.isLabAccountExist,
      wallet: (state) => state.substrate.wallet,
      web3: (state) => state.metamask.web3,
      lastEventData: (state) => state.substrate.lastEventData
    }),

    priceHint() {
      return `${this.document.price} ${this.document.currency} = ${(this.currentDAIprice * this.document.price).toFixed(4)} USD`
    },

    qcPriceHint() {
      return `${this.document.qcPrice} ${this.document.currency} = ${(this.currentDAIprice * this.document.qcPrice).toFixed(4)} USD`
    },

    hasServicePayload() {
      return Boolean(Object.keys(this.servicePayload).length)
    },

    fieldRequiredRule() {
      return [
        val => !!val || "This field is required"
      ]
    },

    serviceNameRules() {
      return [
        val => (val && val.length <= 50) || "This field only allows 50 characters."
      ]
    },

    decimalRule() {
      return [
        val => /^\d*(\.\d{0,4})?$/.test(val) || this.isBiomedical || "This field only allows 4 decimal characters."
      ]
    },

    fieldEnglishRules() {
      return [ englishAlphabet ]
    },

    descriptionRules() {
      return [
        val => (val && val.length <= 100) || "This field only allows 100 characters."
      ]
    },

    longDescriptionRules() {
      return [
        englishAlphabet,
        val => (val && val.length <= 500) || "This field only allows 500 characters."
      ]
    },

    fileInputRules() {
      return [
        value => !Array.isArray(value) || "This field is required",
        value => (!Array.isArray(value) && value?.size < 2000000) || "The total file size uploaded exceeds the maximum file size allowed (2MB)",
        value => (!Array.isArray(value) && value?.type === "application/pdf") || "The files uploaded are not in the supported file formats."
      ]
    }

  },

  async mounted() {
    this.dnaCollectionProcessList = (await getDNACollectionProcess()).data
  },

  async created() {
    this.validate()
    this.prefillValues()
    await this.getServiceCategory()
    const { daiToUsd } = await getConversionCache()
    this.currentDAIprice = daiToUsd ?? 1
  },

  methods: {

    async validate () {
      this.labInfo = await queryLabsById(this.api, this.wallet.address)

      const gitbookLink = `<a href="https://t.me/debionetwork" target="_blank">contact us</a>`

      const MESSAGE = Object.freeze({
        UNVERIFIED: {
          type: "UNVERIFIED",
          actionTitle: "Close",
          title: "Your verification process is still under review",
          subtitle: `
            We're sorry to say that you cannot provide a service until you are verified.
            Please ${gitbookLink} for more infomation
          `
        },
        REJECTED: {
          type: "REJECTED",
          actionTitle: "Close",
          title: "Your verification process is rejected",
          subtitle: `
            We're sorry to say that you cannot provide a service because your verification status is rejected
            Please contact us ${gitbookLink} for more infomation
          `
        },
        REVOKED: {
          type: "REVOKED",
          actionTitle: "Close",
          title: "Your verification process is revoked",
          subtitle: `
            We're sorry to say that you cannot provide a service because your verification status is revoked
            Please contact us ${gitbookLink} for more infomation
          `
        },
        NOT_EXIST: {
          type: "NOT_EXIST",
          actionTitle: "Proceed",
          title: "You are not registered yet",
          subtitle: "Complete your registration process first before continue"
        },
        CITY_NOT_MATCH: {
          type: "CITY_NOT_MATCH",
          actionTitle: "Close",
          title: "Add service failed",
          subtitle: "Your location is not match with the requested service!"
        }
      })

      if (!this.labInfo) {
        this.showModalAlert = true

        this.messageWarning = MESSAGE["NOT_EXIST"]
      }

      if (this.labInfo?.verificationStatus === "Verified" && Object.keys(this.servicePayload).length) {
        if (
          this.labInfo.info?.country !== this.servicePayload.countryCode ||
          this.labInfo.info?.region !== this.servicePayload.cityCode
        ) {

          this.showModalAlert = true

          this.messageWarning = MESSAGE["CITY_NOT_MATCH"]

          await this.$store.dispatch("lab/setProvideService", {})

          return
        }
      }

      this.statusLab = this.labInfo.verificationStatus

      if (this.statusLab === "Verified") return

      const notExist = !this.exist || !this.labInfo.certifications.length || !this.labInfo.services.length

      const compute = notExist ? "NOT_EXIST" : this.labInfo.verificationStatus.toUpperCase()

      this.messageWarning = MESSAGE[compute]

      this.showModalAlert = true
    },

    async getServiceCategory() {
      const { data : data } = await getCategories()
      this.listCategories =  data
    },

    prefillValues() {
      const checkQuery = Object.keys(this.servicePayload).length
      if (!checkQuery) return

      const {
        category,
        currency,
        currencyType,
        serviceFlow
      } = this.servicePayload


      this.document.category = category
      this.document.currency = currency || currencyType
      this.document.serviceFlow = serviceFlow
      this.serviceFlow = serviceFlow
    },

    async getCreateServiceFee() {
      const { category, dnaCollectionProcess, name, currency, price, qcPrice, description, longDescription, duration, durationType } = this.document
      const newService = {
        name,
        pricesByCurrency: [{
          currency,
          totalPrice: await toEther(price + qcPrice),
          priceComponents: [{
            component: "testing_price",
            value: await toEther(price)
          }],
          additionalPrices: [{
            component: "qc_price",
            value: await toEther(qcPrice)
          }]
        }],
        expectedDuration: { duration, durationType },
        category, description, longDescription, dnaCollectionProcess,
        image: this.imageUrl,
        testResultSample: this.testResultSampleUrl
      }

      const fee = await createServiceFee(this.api, this.wallet, newService, this.serviceFlow)
      const checkQuery = Object.keys(this.servicePayload).length
      const txWeight = this.web3.utils.fromWei(String(fee.partialFee), "ether")
      const claimRequestServiceFee = 0.0203 // Static Estimated claimRequestServiceFee

      if (!checkQuery) {
        this.fee = `${Number(txWeight).toFixed(4)} DBIO`
        return
      }
      this.fee = `${(Number(txWeight) + claimRequestServiceFee).toFixed(4)} DBIO`
    },

    async handleCreateService() {
      if(this.isLoading) return // If function already running return.
      if (!this.$refs.addServiceForm.validate()) {
        return
      }

      this.isLoading = true
      try {
        const { category, dnaCollectionProcess, name, currency, price, qcPrice, description, longDescription, duration, durationType } = this.document
        const newService = {
          name,
          pricesByCurrency: [{
            currency,
            totalPrice: await toEther(price + qcPrice),
            priceComponents: [{
              component: "testing_price",
              value: await toEther(price)
            }],
            additionalPrices: [{
              component: "qc_price",
              value: await toEther(qcPrice)
            }]
          }],
          expectedDuration: { duration, durationType },
          category, description, longDescription, dnaCollectionProcess,
          image: this.imageUrl,
          testResultSample: this.testResultSampleUrl
        }

        await createService(
          this.api,
          this.wallet,
          newService,
          this.serviceFlow
        )
      } catch (err) {
        this.isLoading = false
        if (err.message === "1010: Invalid Transaction: Inability to pay some fees , e.g. account balance too low") {
          this.isShowError = true
        }
        console.error(err)
      }
    },

    closeDialog() {
      this.isShowError = false
    },

    async imageUploadEventListener(file) {
      this.isLoading = true
      this.imageUrl = ""
      if (file) {
        if (file.name.lastIndexOf(".") <= 0) {
          return
        }
        const dataFile = await this.setupFileReader(file)

        const result = await uploadFile({
          title: dataFile.name,
          type: dataFile.type,
          size: dataFile.size,
          file: dataFile
        })

        const link = getFileUrl(result.IpfsHash)
        this.imageUrl = link
        this.isLoading = false
      }
    },

    setupFileReader(value) {
      return new Promise((resolve, reject) => {
        const file = value
        const fr = new FileReader()

        fr.onload = async function () {
          resolve(value)
        }

        fr.onerror = reject

        fr.readAsArrayBuffer(file)
      })
    },

    async fileUploadEventListener(file) {
      if (!file || file.size >= 2000000) {
        return
      }
      this.isLoading = true
      this.testResultSampleUrl = ""
      if (file) {
        if (file.name.lastIndexOf(".") <= 0) {
          return
        }
        const dataFile = await this.setupFileReader(file)

        const result = await uploadFile({
          title: dataFile.name,
          type: dataFile.type,
          size: dataFile.size,
          file: dataFile
        })

        const link = getFileUrl(result.IpfsHash)
        this.testResultSampleUrl = link
        this.isLoading = false
      }
    },

    async handleClaimRequest(id) {
      try {
        if (Object.keys(this.servicePayload).length) {
          const dataRequestServices = await getProvideRequestService(this.servicePayload)

          this.isLoading = true
          for (let i=0; i < dataRequestServices.length; i++) {
            await claimRequestService(this.api, this.wallet, {
              id,
              hash: dataRequestServices[i].request.hash,
              testing_price: await toEther(this.document.price),
              qc_price: await toEther(this.document.qcPrice)
            })
          }

          this.isLoading = false
          this.$router.push("/lab/services")
          this.$store.dispatch("lab/setProvideService", {})
        }
      } catch (err) {
        if (err.message === "1010: Invalid Transaction: Inability to pay some fees , e.g. account balance too low") {
          this.isShowError = true
        }
        console.error(err);
        this.isLoading = false
      }
    },

    handleRedirect() {
      const REDIRECT_TO = Object.freeze({
        UNVERIFIED: {
          name: "lab-dashboard"
        },
        REJECTED: {
          name: "lab-dashboard"
        },
        REVOKED: {
          name: "lab-dashboard"
        },
        NOT_EXIST: {
          name: "lab-registration"
        },
        CITY_NOT_MATCH: {
          name: "lab-dashboard-services"
        }
      })

      if (this.messageWarning?.type === "CITY_NOT_MATCH") {
        this.$router.push(REDIRECT_TO["CITY_NOT_MATCH"])

        return
      }

      const notExist = !this.exist || !this.labInfo.certifications.length || !this.labInfo.services.length

      const compute = notExist ? "NOT_EXIST" : this.statusLab.toUpperCase()

      this.$router.push(REDIRECT_TO[compute])
    },

    selectPicture() {
      this.$refs.fileInput.$refs.input.click()
    }
  },

  watch: {
    category() {
      if (this.document.category == "Covid-19") {
        this.isBiomedical = true
        this.document.qcPrice = "0"
      } else {
        this.isBiomedical = false
      }
    },

    lastEventData(val) {
      if (val === null) return
      const dataEvent = JSON.parse(val.data.toString())
      if (val.method === "ServiceCreated") {
        if (dataEvent[1] === this.wallet.address && Object.keys(this.servicePayload).length) this.handleClaimRequest(dataEvent[0].id)

        else this.$router.push("/lab/services")
      }
    },

    document: {
      deep: true,
      immediate: true,
      handler: generalDebounce(async function() {
        await this.getCreateServiceFee()
      }, 500)
    }
  }
}
</script>
