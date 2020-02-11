<template>
  <div id="filing-history-list">
    <DownloadErrorDialog
      :dialog="downloadErrorDialog"
      @close="downloadErrorDialog = false"
      attach="#filing-history-list"
    />

    <v-expansion-panels v-if="filedItems && filedItems.length > 0" v-model="panel">
      <v-expansion-panel
        class="align-items-top filing-history-item"
        v-for="(item, index) in filedItems"
        :key="index"
      >
        <v-expansion-panel-header>
          <div class="list-item">
            <div>
              <h3>{{item.name}}</h3>
              <div class="list-item__subtitle">
                <v-scale-transition v-if="isCoaFutureEffective(item.name, item.status)">
                  <v-tooltip
                    top
                    content-class="pending-tooltip"
                  >
                    <template v-slot:activator="{ on }">
                      <div id="pending-alert" class="list-item__subtitle" v-on="on">
                        <span>
                          FILED AND PENDING (filed by {{item.filingAuthor}} on {{item.filingDate}})
                        </span>
                        <v-icon color="yellow" small>mdi-alert</v-icon>
                      </div>
                    </template>
                    <span>
                      The updated office addresses will be legally effective on {{ item.filingEffectiveDate }},
                      12:01 AM (Pacific Time). No other filings are allowed until then.
                    </span>
                  </v-tooltip>
                </v-scale-transition>
                <span v-else>FILED AND PAID (filed by {{item.filingAuthor}} on {{item.filingDate}})</span>
                <span>2 comments</span>
              </div>
            </div>

            <div class="filing-item__actions">
              <div class="toggle-info mr-3">
                <template v-if="panel === index">
                  <span v-if="item.paperOnly">Close</span>
                  <span v-else>Hide Documents</span>
                </template>
                <template v-else>
                  <span v-if="item.paperOnly">Request a Copy</span>
                  <span v-else>View Documents</span>
                </template>
              </div>
              <v-menu bottom left transition="slide-y-transition">
                <template v-slot:activator="{ on }">
                  <v-btn icon v-on="on">
                    <v-icon>mdi-dots-horizontal</v-icon>
                  </v-btn>
                </template>
                <v-list dense>
                  <v-list-item-group color="primary">
                    <v-list-item>
                      <v-list-item-icon>
                        <v-icon>mdi-file-document-edit-outline</v-icon>
                      </v-list-item-icon>
                      <v-list-item-title>File a Correction</v-list-item-title>
                    </v-list-item>
                    <v-list-item>
                      <v-list-item-icon>
                        <v-icon>mdi-comment-plus</v-icon>
                      </v-list-item-icon>
                      <v-list-item-title @click="commentDialog='true'">Add Comment</v-list-item-title>
                    </v-list-item>
                  </v-list-item-group>
                </v-list>
              </v-menu>
            </div>
          </div>

        </v-expansion-panel-header>

        <v-expansion-panel-content>
          <v-list dense class="mt-n1 mb-n3 pt-0 pb-0" v-if="!item.paperOnly">
            <v-list-item class="pl-0 pr-0"
              v-for="(document, index) in item.filingDocuments"
              :key="index"
            >
              <v-btn text color="primary" class="pl-1 pr-2"
                @click="downloadDocument(document)"
                :disabled="loadingDocument"
                :loading="loadingDocument"
              >
                <v-icon>mdi-file-pdf-outline</v-icon>
                <span>{{document.name}}</span>
              </v-btn>
            </v-list-item>
            <v-list-item class="pl-0 pr-0"
              v-if="item.paymentToken"
            >
              <v-btn text color="primary" class="pl-1 pr-2"
                @click="downloadReceipt(item)"
                :disabled="loadingReceipt"
                :loading="loadingReceipt"
              >
                <v-icon>mdi-file-pdf-outline</v-icon>
                <span>Receipt</span>
              </v-btn>
            </v-list-item>
            <v-list-item class="pl-0 pr-0">
              <v-btn text color="primary" class="pl-1 pr-2"
                @click="downloadAll(item)"
                :disabled="loadingAll"
                :loading="loadingAll"
              >
                <v-icon>mdi-download</v-icon>
                <span>Download all</span>
              </v-btn>
            </v-list-item>
          </v-list>

          <div class="body-2" v-if="item.paperOnly">
            <p>Filings completed <b>before March 10, 2019</b>
            are only available from the BC Registry as paper documents.</p>
            <p>To request copies of paper documents, contact BC Registry Staff
            with the document you require and the name and incorporation number of your association:</p>
            <ul class="contact-info__list mt-5">
              <li>
                <span>Toll Free:</span> 1 877 526-1526
              </li>
              <li>
                <span>Phone:</span> 250 387-7848
              </li>
              <li>
                <span>Email:</span> <a href="mailto:bcregistries@gov.bc.ca">bcregistries@gov.bc.ca</a>
              </li>
            </ul>
          </div>

          <div class="comments-section mt-8">
            <v-divider></v-divider>
            <div class="title-bar mt-5">
              <h4>Comments (2)</h4>
              <v-btn color="primary" @click.stop="commentDialog = true">Add a comment</v-btn>
            </div>
            <div>
              <v-list>
                <v-list-item class="pl-0 pr-0">
                  <v-list-item-content>
                    <v-list-item-title class="body-2">
                      <strong>idir/scotdavi</strong> (2019-09-30 - 12:45pm PST)
                    </v-list-item-title>
                    <v-list-item-subtitle class="body-2">
                      <div>Address Change Correction</div>
                      <div>Changed delivery address</div>
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item class="pl-0 pr-0">
                  <v-list-item-content>
                    <v-list-item-title class="body-2">
                      <strong>idir/scotdavi</strong> (2019-09-30 - 12:45pm PST)
                    </v-list-item-title>
                    <v-list-item-subtitle class="body-2">
                      <div>Address Change Correction</div>
                      <div>Changed mailing address</div>
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </div>
          </div>

        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>

    <!-- No Results Message -->
    <v-card class="no-results" flat v-if="filedItems && filedItems.length === 0">
      <v-card-text>
        <div class="no-results__title">You have no filing history</div>
        <div class="no-results__subtitle">Your completed filings and transactions will appear here</div>
      </v-card-text>
    </v-card>

    <v-dialog width="640" v-model="commentDialog">
      <v-card>
        <v-card-title>Add a comment</v-card-title>
        <v-card-text>
          <v-form>
            <v-textarea dense filled auto-grow rows="2" row-height="26" class="mt-3"
            label="Add a comment"
            hide-details
            ></v-textarea>
            <v-checkbox dense hide-details label="Attach comment to this filing"></v-checkbox>
            <v-checkbox dense hide-details class="mt-1"
              label="Create a separate ledger entry for this comment">
            </v-checkbox>
            <div class="form__btns mt-5" @click.stop="commentDialog = false">
              <v-btn color="primary">Post</v-btn>
            </div>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts">
// Libraries
import axios from '@/axios-auth'
import { mapState } from 'vuex'

// Dialogs
import { DownloadErrorDialog } from '@/components/dialogs'

// Enums
import { EntityTypes, FilingStatus, FilingTypes } from '@/enums'

// Mixins
import DateMixin from '@/mixins/date-mixin'
import EntityFilterMixin from '@/mixins/entityFilter-mixin'

export default {
  name: 'FilingHistoryList',

  mixins: [DateMixin, EntityFilterMixin],

  components: {
    DownloadErrorDialog
  },

  data () {
    return {
      downloadErrorDialog: false,
      panel: null, // currently expanded panel
      filedItems: null,
      loadingDocument: false,
      loadingReceipt: false,
      loadingAll: false,
      commentDialog: false,

      // Enum
      EntityTypes
    }
  },

  computed: {
    ...mapState(['entityIncNo', 'filings', 'entityName'])
  },

  created () {
    // load data into this page
    this.loadData()
  },

  methods: {
    loadData () {
      this.filedItems = []

      // create filed items
      for (let i = 0; i < this.filings.length; i++) {
        const filing = this.filings[i].filing
        if (filing && filing.header) {
          let filingDate = filing.header.date.slice(0, 10)
          if (filingDate < '2019-03-08' || filing.header.availableOnPaperOnly) {
            this.loadPaperFiling(filing)
          } else {
            switch (filing.header.name) {
              case 'annualReport':
                this.loadAnnualReport(filing)
                break
              case 'changeOfDirectors':
                this.loadReport(FilingTypes.DIRECTOR_CHANGE, filing, filing.changeOfDirectors)
                break
              case 'changeOfAddress':
                this.loadReport(FilingTypes.ADDRESS_CHANGE, filing, filing.changeOfAddress)
                break
              case 'changeOfName':
                this.loadReport(FilingTypes.LEGAL_NAME_CHANGE, filing, filing.changeOfName)
                break
              case 'specialResolution':
                this.loadReport(FilingTypes.SPECIAL_RESOLUTION, filing, filing.specialResolution)
                break
              case 'voluntaryDissolution':
                this.loadReport(FilingTypes.VOLUNTARY_DISSOLUTION, filing, filing.voluntaryDissolution)
                break
              default:
                this.loadPaperFiling(filing)
                break
            }
          }
        } else {
          // eslint-disable-next-line no-console
          console.log('ERROR - invalid filing or filing header =', filing)
        }
      }

      this.$emit('filed-count', this.filedItems.length)
      this.$emit('filings-list', this.filedItems)

      // if needed, highlight a specific filing
      // NB: use unary plus operator to cast string to number
      const highlightId = +this.$route.query.filing_id // may be NaN (which is false)
      if (highlightId) { this.highlightFiling(highlightId) }
    },

    // Method to extract date from a local datetime string
    // Returns "yyyy-mm-dd"
    formatDate (dateString): string {
      if (!dateString) return null // safety check
      return dateString.split(' ')[0]
    },

    loadAnnualReport (filing) {
      if (filing.annualReport) {
        const date = filing.annualReport.annualReportDate
        const localDateTime = this.convertUTCTimeToLocalTime(filing.header.date)
        const filingDate = this.formatDate(localDateTime)
        if (date) {
          const agmYear = +date.substring(0, 4)
          const item = {
            name: `Annual Report (${agmYear})`,
            filingId: filing.header.filingId,
            filingAuthor: filing.header.certifiedBy,
            filingDateTime: localDateTime,
            filingDate: filingDate,
            paymentToken: filing.header.paymentToken,
            filingDocuments: [{
              filingId: filing.header.filingId,
              name: 'Annual Report',
              documentName: `${this.entityIncNo} - Annual Report (${agmYear}) - ${filingDate}.pdf`
            }],
            paperOnly: false
          }
          this.filedItems.push(item)
        } else {
          // eslint-disable-next-line no-console
          console.log('ERROR - invalid date in filing =', filing)
        }
      } else {
        // eslint-disable-next-line no-console
        console.log('ERROR - invalid annualReport in filing =', filing)
      }
    },

    loadReport (title, filing, section) {
      if (section) {
        const localDateTime = this.convertUTCTimeToLocalTime(filing.header.date)
        const filingDate = this.formatDate(localDateTime)

        // Effective date is when the filing is applied to the backend, assigned by the backend when the filing is made.
        // Currently, all filings will be applied immediately except a change of address for a Benefit Company
        // The latter is always effective the following day at 12:01 AM Pacific Time
        let effectiveDate = filing.header.date.slice(0, 10)
        if (filing.header.effectiveDate) effectiveDate = filing.header.effectiveDate.slice(0, 10)

        const item = {
          name: title,
          filingId: filing.header.filingId,
          filingAuthor: filing.header.certifiedBy,
          filingDateTime: localDateTime,
          filingDate: filingDate,
          filingEffectiveDate: effectiveDate,
          paymentToken: filing.header.paymentToken,
          filingDocuments: [{
            filingId: filing.header.filingId,
            name: title,
            documentName: `${this.entityIncNo} - ${title} - ${filingDate}.pdf`
          }],
          status: filing.header.status,
          paperOnly: false
        }
        this.filedItems.push(item)
      } else {
        // eslint-disable-next-line no-console
        console.log(`ERROR - invalid ${title} in filing =`, filing)
      }
    },

    loadPaperFiling (filing) {
      // split name on camelcase and capitalize first letters
      if (!filing || !filing.header || !filing.header.name) return // safety check
      let name = filing.header.name.split(/(?=[A-Z])/).join(' ')
      const localDateTime = this.convertUTCTimeToLocalTime(filing.header.date)
      const filingDate = this.formatDate(localDateTime)
      name = name.charAt(0).toLocaleUpperCase() + name.slice(1)
      const item = {
        name: name,
        filingId: filing.header.filingId,
        filingAuthor: 'Registry Staff',
        filingDate: filingDate,
        filingYear: filing.header.date.slice(0, 4),
        paymentToken: null,
        filingDocuments: [{
          filingId: filing.header.filingId,
          name: name,
          documentName: null
        }],
        paperOnly: true
      }
      this.filedItems.push(item)
    },

    highlightFiling (highlightId) {
      // expand the panel of the matching filing
      for (let i = 0; i < this.filedItems.length; i++) {
        // assume there is always a filing document
        if (this.filedItems[i].filingDocuments[0].filingId === highlightId) {
          this.panel = i
          break
        }
      }
    },

    async downloadDocument (filingDocument) {
      this.loadingDocument = true
      await this.downloadOneDocument(filingDocument)
      this.loadingDocument = false
    },

    async downloadOneDocument (filingDocument) {
      const url = this.entityIncNo + '/filings/' + filingDocument.filingId
      const headers = { 'Accept': 'application/pdf' }

      await axios.get(url, { headers: headers, responseType: 'blob' as 'json' }).then(response => {
        if (response) {
          /* solution from https://github.com/axios/axios/issues/1392 */

          // it is necessary to create a new blob object with mime-type explicitly set
          // otherwise only Chrome works like it should
          const blob = new Blob([response.data], { type: 'application/pdf' })

          // IE doesn't allow using a blob object directly as link href
          // instead it is necessary to use msSaveOrOpenBlob
          if (window.navigator && window.navigator.msSaveOrOpenBlob) {
            window.navigator.msSaveOrOpenBlob(blob, filingDocument.documentName)
          } else {
            // for other browsers, create a link pointing to the ObjectURL containing the blob
            const url = window.URL.createObjectURL(blob)
            const a = window.document.createElement('a')
            window.document.body.appendChild(a)
            a.setAttribute('style', 'display: none')
            a.href = url
            a.download = filingDocument.documentName
            a.click()
            window.URL.revokeObjectURL(url)
            a.remove()
          }
        } else {
          // eslint-disable-next-line no-console
          console.log('downloadOneDocument() error - null response')
          this.downloadErrorDialog = true
        }
      }).catch(error => {
        // eslint-disable-next-line no-console
        console.error('loadOneDocument() error =', error)
        this.downloadErrorDialog = true
      })
    },

    async downloadReceipt (filing) {
      this.loadingReceipt = true
      await this.downloadOneReceipt(filing)
      this.loadingReceipt = false
    },

    async downloadOneReceipt (filing) {
      const url = filing.paymentToken + '/receipts'
      const data = {
        corpName: this.entityName,
        filingDateTime: filing.filingDateTime, // TODO: format as needed
        fileName: 'receipt' // not used
      }
      const config = {
        headers: { 'Accept': 'application/pdf' },
        responseType: 'blob' as 'json',
        baseURL: sessionStorage.getItem('PAY_API_URL') + 'payment-requests/'
      }

      await axios.post(url, data, config).then(response => {
        if (response) {
          const fileName = `${this.entityIncNo} - Receipt - ${filing.filingDate}.pdf`

          /* solution from https://github.com/axios/axios/issues/1392 */

          // it is necessary to create a new blob object with mime-type explicitly set
          // otherwise only Chrome works like it should
          const blob = new Blob([response.data], { type: 'application/pdf' })

          // IE doesn't allow using a blob object directly as link href
          // instead it is necessary to use msSaveOrOpenBlob
          if (window.navigator && window.navigator.msSaveOrOpenBlob) {
            window.navigator.msSaveOrOpenBlob(blob, fileName)
          } else {
            // for other browsers, create a link pointing to the ObjectURL containing the blob
            const url = window.URL.createObjectURL(blob)
            const a = window.document.createElement('a')
            window.document.body.appendChild(a)
            a.setAttribute('style', 'display: none')
            a.href = url
            a.download = fileName
            a.click()
            window.URL.revokeObjectURL(url)
            a.remove()
          }
        } else {
          // eslint-disable-next-line no-console
          console.log('downloadOneReceipt() error - null response')
          this.downloadErrorDialog = true
        }
      }).catch(error => {
        // eslint-disable-next-line no-console
        console.error('downloadOneReceipt() error =', error)
        this.downloadErrorDialog = true
      })
    },

    async downloadAll (filing) {
      this.loadingAll = true
      // first download document(s)
      for (let i = 0; i < filing.filingDocuments.length; i++) {
        await this.downloadOneDocument(filing.filingDocuments[i])
      }
      // finally download receipt
      if (filing.paymentToken) {
        await this.downloadOneReceipt(filing)
      }
      this.loadingAll = false
    },

    /**
     * Function to return a boolean if this specific filing is future affective.
     *
     * @param filingType The type of the filing in the history list.
     * @param status The status of the filing in the history list.
     * @return A boolean indicating if the filing is future effective.
     */
    isCoaFutureEffective (filingType: string, status: string): boolean {
      return this.entityFilter(EntityTypes.BCOMP) &&
        filingType === FilingTypes.ADDRESS_CHANGE &&
        status === FilingStatus.PAID
    }

  },

  watch: {
    filings () {
      // if filings changes, reload them
      // (does not fire on initial page load)
      this.loadData()
    }
  }
}
</script>

<style lang="scss" scoped>
@import "@/assets/styles/theme.scss";

.list-item {
  align-items: flex-start;
  justify-content: space-between;
  padding: 0;
}

.v-col-padding {
  padding: 0 12px 0 12px;
}

.filing-label {
  flex-basis: 33.3333%;
  flex: 1 1 auto;
}

.filing-action {
  flex: 0 0 auto;
  width: 30%;
  text-align: right;
  font-weight: 700;
}

.pending-tooltip {
  max-width: 16rem;
}

.pending-icon {
  background-color: black;
}

// Past Filings
.past-filings {
  border-top: 1px solid $gray3;
  text-align: center;

  .past-filings__text {
    margin-top: 0.25rem;
    color: $gray6;
    font-size: 0.875rem;
    font-weight: 500;
  }
}

.v-textarea textarea {
  line-height: 1.5rem;
}

.list-item__subtitle {
  span + span {
    &:before {
      display: inline-block;
      margin-left: 0.5rem;
      margin-right: 0.5rem;
      content: '•'
    }
  }
}

.form__btns {
  justify-content: flex-end;
}

.v-expansion-panel-header {
  padding-top: 1.25rem !important;
  padding-bottom: 1.25rem !important;
}

.title-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

::v-deep {
  .v-expansion-panel-header__icon {
    display: none !important;
  }
}

.filing-history-item {
  h3 {
    margin-bottom: 0.25rem;
  }
}

.filing-item__actions {
  margin-top: -0.5rem;
  white-space: nowrap;

  .toggle-info {
    display: inline-block;
    letter-spacing: -0.01rem;
    font-size: 0.875rem;
    font-weight: 700;
  }
}

.comments-section {
  h4 {
    letter-spacing: 0;
    font-size: 0.9375rem;
    font-weight: 700;
  }
}
</style>
