<template>
  <div id="uploadDocument">
    <HeaderOptions />
    <v-row class="mt-4 ml-4">
      <p class="fontStyle font-weight-bold mb-6 mt-6 ml-0">
        Documents Checklist
        <span class="orangetext">({{ totalListCount }})</span>
      </p>
      <v-col align="right" class="mb-0 mt-3 mr-5">
        <v-row class="rowMargin mt-0" v-if="Documents.length != 0">
          <span
            class="textcolr font-weight-bold mr-2"
            @click="viewRepository()"
          >
            View Document Repository
          </span>
          <span>
            <v-checkbox
              class="workitemcheck checkmargin"
              :disabled="allDocumentsReceived"
              v-model="allDocumentsReceived"
              color="#23B1A9"
              label="All Documents Recieved"
              @click="shwRemarkDialog()"
            />
          </span>
        </v-row>
      </v-col>
    </v-row>

    <template>
      <v-data-table
        :headers="doc_headers"
        :items="Documents"
        disable-pagination
        hide-default-footer
        :expanded.sync="expanded"
        :single-expand="singleExpand"
        class="vDataTableStyling docborder documentable mrgncls mx-3 mt-0 uploadDocument__custom__table"
        no-data-text="No Records for Documents"
      >
        <template v-slot:[`item.main.documentName`]="{ item }">
          <v-tooltip v-model="item.documentNametip" top>
            <template v-slot:activator="{ on, attrs }">
              <div v-on="on" v-bind="attrs">
                <span v-if="item.main.isMandatory" class="mandatorycolor">
                  * </span
                >{{ item.main.documentName.slice(0, 20) }}
              </div>
            </template>
            <span>{{ item.main.documentName }}</span>
          </v-tooltip>
        </template>
        <template v-slot:[`item.action`]="{ item, index }">
          <v-row>
            <v-tooltip v-model="item.uploadtip" top>
              <template v-slot:activator="{ on, attrs }">
                <div
                  @mouseenter="item.uploadtip = true"
                  @mouseleave="item.uploadtip = false"
                >
                  <v-icon
                    class="iconsize mr-2"
                    color="#23B1A9"
                    @click="expandRow(item, index, 'upload')"
                    v-on="on"
                    v-bind="attrs"
                    >mdi-cloud-upload</v-icon
                  >
                </div>
              </template>
              <span>{{ "Upload" }}</span>
            </v-tooltip>
            <v-tooltip v-model="item.viewtip" top>
              <template v-slot:activator="{ on, attrs }">
                <div
                  @mouseenter="item.viewtip = true"
                  @mouseleave="item.viewtip = false"
                >
                  <v-icon
                    class="iconsize mr-2"
                    color="#01bbd9"
                    @click="expandRow(item, index, 'view')"
                    v-on="on"
                    v-bind="attrs"
                    >mdi-eye-outline{{ item.action }}</v-icon
                  >
                </div>
              </template>
              <span>{{ "Previous Documents" }}</span>
            </v-tooltip>
            <v-tooltip v-model="item.historytip" top>
              <template v-slot:activator="{ on, attrs }">
                <div
                  @mouseenter="item.historytip = true"
                  @mouseleave="item.historytip = false"
                >
                  <v-icon
                    class="iconsize"
                    color="orange"
                    @click="expandRow(item, index, 'history')"
                    v-on="on"
                    v-bind="attrs"
                    >mdi-restore</v-icon
                  >
                </div>
              </template>
              <span>{{ "History" }}</span>
            </v-tooltip>
          </v-row>
        </template>
        <template v-slot:[`item.main.documentRequested`]="{ item }">
          <v-select
            v-model="item.main.documentRequested"
            solo
            class="forminput textfieldsize"
            :items="reqArr"
            item-text="text"
            :disabled="item.main.isMandatory"
            dense
          >
          </v-select>
        </template>
        <template v-slot:[`item.main.dateRequested`]="{ item }">
          <v-menu
            class="fnolReportedDate"
            v-model="item.main.dateReqMenu"
            :close-on-content-click="false"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on }">
              <v-text-field
                placeholder="DD/MM/YYYY"
                dense
                solo
                readonly
                :value="
                  item.main.isMandatory
                    ? formatDate(item.main.createdAt)
                    : formatDate(item.main.dateRequested)
                "
                class="forminput textfieldsize"
                v-on="on"
              >
              </v-text-field>
            </template>
          </v-menu>
        </template>
        <template v-slot:[`item.main.dateRecieved`]="{ item }">
          <v-menu
            class="fnolReportedDate"
            v-model="item.main.dateRecMenu"
            :close-on-content-click="false"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on }">
              <v-text-field
                placeholder="DD/MM/YYYY"
                dense
                solo
                readonly
                :value="formatDate(item.main.dateRecieved)"
                class="forminput textfieldsize"
                v-on="on"
              >
              </v-text-field>
            </template>
            <v-date-picker
              no-title
              :max="todayDate"
              :min="modifiedreqdate"
              v-model="item.main.dateRecieved"
              @input="item.main.dateRecMenu = false"
            >
            </v-date-picker>
          </v-menu>
        </template>
        <template v-slot:[`item.main.documentStatus`]="{ item }">
          <v-select
            v-model="item.main.documentStatus"
            solo
            class="forminput textfieldsize"
            :items="StatusArray"
            dense
          >
          </v-select>
        </template>
        <template v-slot:[`item.main.waiverReason`]="{ item }">
          <v-tooltip :disabled="!item.main.waiverReason" top>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="item.main.waiverReason"
                solo
                v-on="on"
                v-bind="attrs"
                placeholder="Enter Reason"
                :disabled="item.main.documentStatus != 'Waived'"
                class="forminput textfieldsize"
                dense
              >
              </v-text-field>
            </template>
            <span>{{ item.main.waiverReason }}</span>
          </v-tooltip>
        </template>
        <template v-slot:[`item.main.remarks`]="{ item }">
          <v-tooltip :disabled="!item.main.remarks" top>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="item.main.remarks"
                solo
                v-on="on"
                v-bind="attrs"
                placeholder="Enter Remarks"
                class="forminput textfieldsize"
                dense
              >
              </v-text-field>
            </template>
            <span>{{ item.main.remarks }}</span>
          </v-tooltip>
        </template>
        <template v-slot:[`item.Save`]="{ item }">
          <v-btn class="savebtn" @click="save(item)">Save</v-btn>
        </template>
        <template v-slot:[`item.main.isMandatory`]="{ item }">
          <span>{{ item.main.isMandatory == true ? "Yes" : "No" }}</span>
        </template>
        <template v-slot:[`item.main.isSubmitted`]="{ item }">
          <span>{{ item.main.isSubmitted == true ? "Yes" : "No" }}</span>
        </template>
        <template v-slot:[`item.main.createdAt`]="{ item }">
          <span>{{
            item.main.createdAt ? dateFormat(item.main.createdAt) : "------"
          }}</span>
        </template>
        <template v-slot:expanded-item="{ headers, item }" :scope="props">
          <td
            class="px-1"
            :colspan="headers.length"
            v-if="
              item.docShow.upload || item.docShow.view || item.docShow.history
            "
          >
            <!-- upload -->
            <div v-if="item.docShow.upload">
              <v-row class="mt-2 ml-4">
                <p class="font-weight-bold">Upload Required Documents</p>
              </v-row>
              <v-layout>
                <v-flex
                  class="uploadbr mb-5 ml-4"
                  @dragover="dragover"
                  @dragleave="dragleave"
                  @drop="drop($event, item.index)"
                  :class="visualEffect ? 'dragbackground' : ''"
                >
                  <v-icon color="#23b1a9" class="iconfont"
                    >mdi-tray-arrow-up</v-icon
                  ><span class="ml-2 browsetext font-weight-bold"
                    >Drop Files To Upload or
                    <a @click="browse(item.index)"><u> Browse</u></a></span
                  >
                  <p class="browsetext margincls dlt_text mb-0">
                    Max 5MB of PDF or Image(.jpeg, .jpg, .png)
                  </p>
                  <p class="dlt_text mt-0 ml-4" style="font-size: 10px">
                    Multiple documents can be uploaded
                  </p>
                  <span></span>
                  <v-file-input
                    multiple
                    v-model="all_attachments"
                    @change="uploadFile(item.index, $event)"
                    id="file_upload"
                    style="display: none"
                  >
                  </v-file-input>
                </v-flex>
                <v-flex
                  class="uploadbr mb-5 ml-6"
                  @click="browseRepo(item.index)"
                >
                  <v-icon color="#23b1a9" class="iconfont"
                    >mdi-tray-arrow-up</v-icon
                  ><span class="ml-2 browsetext font-weight-bold"
                    >Click Here To Upload From Repository</span
                  >
                  <p class="browsetext margincls dlt_text mb-0 ml-3">
                    Documents uploaded in Job Order or Purchase
                  </p>
                  <p class="dlt_text mt-0 ml-4">Order</p>
                  <span></span>
                  <v-file-input
                    multiple
                    v-model="all_repo"
                    @change="uploadFromRepo()"
                    accept=".doc, .docx, .csv, image/png,image/jpeg,application/msword,application/pdf,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.openxmlformats-officedocument.wordprocessingml.document, text/csv"
                    id="upload_doc"
                    style="display: none"
                  >
                  </v-file-input>
                </v-flex>

                <v-col cols="4" class="ml-2 pr-0 pt-0">
                  <v-data-table
                    :headers="attachmentheaders"
                    :items="tableDocData[item.index]"
                    hide-default-footer
                    no-data-text="No Records for Attachements"
                    class="attach mt-0 tableborder attch_width"
                  >
                    <template v-slot:[`item.action`]="{ item, tableindex }">
                      <v-icon
                        class="tableicons mr-4"
                        color="#23B1A9"
                        @click="viewDocument(item)"
                        >mdi-eye</v-icon
                      >
                      <v-icon
                        class="tableicons"
                        color="#D1121B"
                        @click="deleteDocument(tableindex)"
                        >mdi-delete</v-icon
                      >
                    </template>
                    {{ tableDocData }}
                  </v-data-table>
                </v-col>
              </v-layout>
            </div>

            <!-- eye -->
            <div v-if="item.docShow.view" class="mt-4">
              <v-row>
                <span class="font-weight-bold ml-7 mb-1 mt-2"
                  >Document List</span
                >
                <v-col align="right" class="mb-0 mt-0 mr-5 closediv pb-3">
                  <span @click="closeExpand()">CLOSE</span>
                </v-col>
              </v-row>
              <v-data-table
                class="vDataTableStyling tablewidth docborder mx-3 mb-3 mt-0"
                hide-default-footer
                no-data-text="No Records for Documents"
                :headers="docListHeaders"
                :items="item.main.document"
              >
                <template v-slot:[`item.file`]="{ item }">
                  <span
                    class="textcolr font-weight-bold"
                    @click="viewDocument(item)"
                    >{{ item.file }}</span
                  >
                </template>
              </v-data-table>
            </div>

            <!-- history -->
            <div v-if="item.docShow.history" class="mt-4">
              <v-flex class="ml-3 mr-3 mb-3">
                <v-row class="groupRowHeadNewLable mt-0">
                  <v-col align="left" class="mt-0 pt-2">
                    <v-flex class="ml-0 GroupNameNewLable">History</v-flex>
                  </v-col>
                  <v-col align="right" class="pt-1 mt-0">
                    <v-icon
                      class="mr-2 viconfont"
                      dark
                      @click="doc_history = !doc_history"
                    >
                      {{
                        doc_history == true ? "mdi-minus-box" : "mdi-plus-box"
                      }}
                    </v-icon>
                  </v-col>
                </v-row>
              </v-flex>

              <v-flex v-if="doc_history">
                <template>
                  <v-simple-table
                    class="vDataTableStyling docborder mx-0 mb-8 mt-0"
                    hide-default-footer
                  >
                    <template>
                      <thead>
                        <tr>
                          <th class="text-center" width="8%">Document Name</th>
                          <th class="text-center" width="15%">Documents</th>
                          <th class="text-center" width="6%">Is Mandatory</th>
                          <th class="text-center" width="6%">Submitted</th>
                          <th class="text-center" width="8%">
                            Document Requested
                          </th>
                          <th class="text-center" width="10%">
                            Date Requested
                          </th>
                          <th class="text-center" width="10%">Date Recieved</th>
                          <th class="text-center" width="8%">
                            Document Status
                          </th>
                          <th class="text-center" width="10%">Waive Reason</th>
                          <th class="text-center" width="10%">Remarks</th>
                          <th class="text-center" width="15%">Date/Time</th>
                          <th class="text-center" width="10%">NTID</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr
                          v-for="(item, Rindex) in item.audit"
                          :key="'CR' + Rindex"
                        >
                          <td>
                            <span>{{
                              item.documentName == ""
                                ? "------"
                                : item.documentName
                            }}</span>
                          </td>
                          <td>
                            <tr
                              v-for="(itm, index) in item.document"
                              :key="'FR' + index"
                            >
                              <td
                                class="textcolr font-weight-bold"
                                style="border: non; hight: 10%"
                                @click="viewDocument(itm)"
                              >
                                {{ itm.file == "" ? "------" : itm.file }}
                              </td>
                            </tr>
                          </td>
                          <td>
                            <span>{{
                              item.isMandatory == true ? "Yes" : "No"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.isSubmitted == true ? "Yes" : "No"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.documentRequested == true ? "Yes" : "No"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.dateRequested
                                ? dateFormat(item.dateRequested)
                                : "------"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.dateRecieved
                                ? dateFormat(item.dateRecieved)
                                : "------"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.documentStatus == ""
                                ? "------"
                                : item.documentStatus
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.waiverReason == ""
                                ? "------"
                                : item.waiverReason
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.remarks == "" ? "------" : item.remarks
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.createdAt
                                ? dateFormat(item.createdAt)
                                : "------"
                            }}</span>
                          </td>
                          <td>
                            <span>{{
                              item.user.employeeName == ""
                                ? "------"
                                : item.user.employeeName
                            }}</span>
                          </td>
                        </tr>
                        <tr
                          class="v-data-table__empty-wrapper"
                          v-if="item.audit.length === 0"
                        >
                          <td colspan="12">No Records For History</td>
                        </tr>
                      </tbody>
                    </template>
                  </v-simple-table>
                </template>
              </v-flex>
            </div>
          </td>
        </template>
      </v-data-table>
    </template>
    
    <UploadRepositoryDocumentsDialog
      v-model="repo_dialog"
    />

    <RemarksDialog 
      v-model="showRemarksDialog"
      @submit="onSubmitRemarks"
    />

  </div>
</template>

<script>
import moment from "moment";
import HeaderOptions from "@/components/HeaderOptions.vue";
import RemarksDialog from "@/components/ViewUploadDocuments/RemarksDialog.vue";
import UploadRepositoryDocumentsDialog from "@/components/ViewUploadDocuments/UploadRepositoryDocumentsDialog.vue";

export default {
  data() {
    return {
      storeValues: {},
      showRemarksDialog: false,

      uploadtip: false,
      viewtip: false,
      historytip: false,
      documentNametip: false,
      remarkstip: false,
      todayDate: moment().format("YYYY-MM-DD"),

      doc_headers: [
        {
          text: "Document Name",
          align: "start",
          value: "main.documentName",
          sortable: false,
        },
        {
          text: "Action",
          align: "start",
          value: "action",
          sortable: false,
          width: "78px",
        },
        {
          text: "Document Requested",
          align: "start",
          value: "main.documentRequested",
          sortable: false,
        },
        {
          text: "Date Requested",
          align: "start",
          value: "main.dateRequested",
          sortable: false,
        },
        {
          text: "Date Received",
          align: "start",
          value: "main.dateRecieved",
          sortable: false,
        },
        {
          text: "Document Status",
          align: "start",
          value: "main.documentStatus",
          sortable: false,
        },
        {
          text: "Waive Reason",
          align: "start",
          value: "main.waiverReason",
          sortable: false,
          width: "100px",
        },
        {
          text: "Remarks",
          align: "start",
          value: "main.remarks",
          sortable: false,
          width: "100px",
        },
        {
          text: "Date/Time",
          align: "start",
          value: "main.createdAt",
          sortable: false,
          width: "100px",
        },
        {
          text: "NTID",
          align: "start",
          value: "main.user.employeeName",
          sortable: false,
        },
        { text: "Action", align: "start", value: "Save", sortable: false },
      ],
      Documents: [],
      expanded: [],
      singleExpand: true,
      StatusArray: ["Received", "Not Received", "Waived"],
      reqArr: [
        { text: "Yes", value: true },
        { text: "No", value: false },
      ],

      attachmentheaders: [
        {
          text: "Document Name",
          align: "start",
          value: "file",
          sortable: false,
        },
        { text: "Action", align: "start", value: "action", sortable: false },
      ],
      all_attachments: [],
      all_repo: [],
      total_uploadedList: [],

      docListHeaders: [
        {
          text: "Document Name",
          align: "start",
          value: "file",
          sortable: false,
        },
        {
          text: "File Type",
          align: "start",
          value: "documentType",
          sortable: false,
        },
      ],

      doc_historyHeaders: [
        {
          text: "Document Name",
          align: "start",
          value: "documentName",
          sortable: false,
        },
        {
          text: "Is Mandatory",
          align: "start",
          value: "isMandatory",
          sortable: false,
        },
        {
          text: "Submitted",
          align: "start",
          value: "isSubmitted",
          sortable: false,
        },
        {
          text: "Document Requested",
          align: "start",
          value: "documentRequested",
          sortable: false,
        },
        {
          text: "Date Requested",
          align: "start",
          value: "dateRequested",
          sortable: false,
        },
        {
          text: "Date Received",
          align: "start",
          value: "dateRecieved",
          sortable: false,
        },
        {
          text: "Document Status",
          align: "start",
          value: "documentStatus",
          sortable: false,
        },
        {
          text: "Waive Reason",
          align: "start",
          value: "waiverReason",
          sortable: false,
        },
        { text: "Remarks", align: "start", value: "remarks", sortable: false },
        {
          text: "Date/Time",
          align: "start",
          value: "createdAt",
          sortable: false,
        },
        {
          text: "NTID",
          align: "start",
          value: "user.employeeName",
          sortable: false,
        },
      ],
      doc_history: true,
      repo_dialog: false,
      repo_headers: [
        { text: "Select", align: "start", value: "select", sortable: false },
        {
          text: "Document Name",
          align: "start",
          value: "file",
          sortable: false,
        },
        {
          text: "Uploaded By",
          align: "start",
          value: "uploadedBy",
          sortable: false,
        },
        {
          text: "Uploaded Date",
          align: "start",
          value: "uploadedDate",
          sortable: false,
        },
      ],
      repoItems: [],
      repo_details: [],
      selectedRepoFiles: [],
      activeIndex: 0,
      tableDocData: [],

      page: 1,
      pageStartIndex: 1,
      pageRowCount: 10,
      totalListCount: 0,
      toolsIndex: null,

      dateRecieved: null,
      dateRecMenu: false,
      dateReqMenu: false,
      dateReq: null,

      claimNumber: "",
      repoIndex: 0,
      featureNumber: "",
      modifiedreqdate: null,
      visualEffect: false, // for showing background color in drag & drop,
      allDocumentsReceived: false,
      phaseFlag: "",
    };
  },

  created() {
    if (this.$store.getters.fetchAllPageValues) {
      this.storeValues = this.$store.getters.fetchAllPageValues;
      this.claimNumber = this.storeValues.claimNumber;
      this.featureNumber = this.storeValues.featureNumber;
      this.phaseFlag = this.storeValues.phaseFlag;
      this.getDocs();
    }
  },

  methods: {
    viewRepository() {
      this.$router.push({
        name: "docRepository",
        params: {
          claimNumber: this.claimNumber,
          featureNumber: this.featureNumber,
          phaseFlag: this.phaseFlag,
        },
      });
    },

    formatDate(date) {
      return date == null ? "" : moment(date).format("DD/MM/YYYY");
    },

    expandRow(item, index, value) {
      this.expanded = [];
      this.expanded = item === this.expanded[index] ? [] : [item];
      let preIndex = -1,
        isMinimised = false;
      for (let i in this.Documents) {
        if (i != index) {
          this.Documents[i].docShow = {
            upload: false,
            view: false,
            history: false,
          };
        }
      }
      console.log("---", this.Documents, index, preIndex);
      if (value == "upload") {
        this.Documents[index].docShow.view = false;
        this.Documents[index].docShow.history = false;
        this.Documents[index].docShow.upload =
          !this.Documents[index].docShow.upload;
      } else if (value == "view") {
        this.Documents[index].docShow.history = false;
        this.Documents[index].docShow.upload = false;
        this.Documents[index].docShow.view =
          !this.Documents[index].docShow.view;
      } else {
        this.Documents[index].docShow.view = false;
        this.Documents[index].docShow.upload = false;
        this.Documents[index].docShow.history =
          !this.Documents[index].docShow.history;
      }
      this.Documents = [...this.Documents];
    },

    checkReqDate(date) {
      this.modifiedreqdate = date;
    },

    closeExpand() {
      this.expanded = [];
    },

    getItem(item) {
      if (item.jobId) {
        return "Job Order Id - " + item.jobId;
      } else if (item.purchaseId) {
        return "Purchase Order Id - " + item.purchaseId;
      } else {
        return "Others";
      }
    },

    dragover(event) {
      event.preventDefault();
      this.visualEffect = true;
    },

    dragleave(event) {
      this.visualEffect = false;
    },

    drop(event, index) {
      event.preventDefault();
      this.all_attachments = event.dataTransfer.files;
      this.uploadFile(index, event.dataTransfer.files); // Trigger the onChange event manually
      this.visualEffect = false;
    },

    browse(index) {
      //to select attachments
      document.getElementById("file_upload").click();
      this.repoIndex = index;
    },

    validateFileType(item) {
      let fileTypeArray = [
        "text/xml", // for .xml
        "application/vnd.openxmlformats-officedocument.presentationml.presentation", // for .pptx
        "application/vnd.ms-powerpoint", // for .ppt
        "text/plain", // for .txt
        "image/jpeg", // for .jpeg
        "image/tiff", // for .tif
        "message/rfc822", // for .eml
        "application/zip", // for .zip
        "application/x-zip-compressed", //compressed zip
        "application/pdf", // for .pdf
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document", // for .docx
        "application/msword", // for .doc
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", // for .xlsx
        "application/vnd.ms-excel", // for .xls
        "image/png", // for .png
        "application/vnd.ms-excel.sheet.macroEnabled.12", // for .xlsm
        //.psd type starts
        "image/photoshop",
        "image/x-photoshop",
        "image/psd",
        "application/photoshop",
        "application/psd",
        "zz-application/zz-winassoc-psd",
        //.psd type ends
      ];
      let foundFileType = false;
      let fileExtension = item.name.substr(item.name.lastIndexOf(".") + 1);
      if (item.type == "" && fileExtension == "msg") {
        foundFileType = true;
      } else {
        foundFileType = fileTypeArray.includes(item.type);
      }
      if (!foundFileType) {
        let file = "";
        if (item.name.includes(".")) {
          file = fileExtension;
        }
        this.showAlert(`${file} file type is not allowed`);
        return false;
      }
      return true;
    },

    uploadFile(index, item) {
      console.log("file typeeeee", item[0]);
      if (!this.validateFileType(item[0])) return;
      if (this.all_attachments.length !== 0) {
        const fsize = this.all_attachments[0].size;
        const file = Math.round(fsize / 1024);
        let self = this;
        if (file > 5120) {
          self.showAlert("File is too big, please select a file less than 5MB");
        } else {
          self.readImage(
            self.all_attachments[0].name,
            self.all_attachments[0],
            (res) => {
              if (this.tableDocData[index].length < 10) {
                this.tableDocData[index].push({
                  file: res.fileName,
                  fileBase: res.base,
                  documentType: self.all_attachments[0].type,
                });

                console.log(this.tableDocData[index]);
              } else {
                this.tableDocData[index].splice(0, 1);
                this.tableDocData[index].push({
                  file: res.fileName,
                  fileBase: res.base,
                  documentType: self.all_attachments[0].type,
                });
              }
            }
          );
        }
      }
    },

    viewDocUsingBase(base64) {
      const base64ImageData = base64;
      const contentType = base64.split(":")[1].split(";")[0];
      const byteCharacters = window.atob(
        base64ImageData.substr(`data:${contentType};base64,`.length)
      );
      const byteArrays = [];
      for (let offset = 0; offset < byteCharacters.length; offset += 1024) {
        const slice = byteCharacters.slice(offset, offset + 1024);
        const byteNumbers = new Array(slice.length);
        for (let i = 0; i < slice.length; i++) {
          byteNumbers[i] = slice.charCodeAt(i);
        }
        const byteArray = new Uint8Array(byteNumbers);
        byteArrays.push(byteArray);
      }
      const blob = new Blob(byteArrays, { type: contentType });
      const blobUrl = window.URL.createObjectURL(blob);
      window.open(blobUrl, "_blank");
    },

    viewDocument(itemData) {
      if (itemData.fileBase) {
        this.viewDocUsingBase(itemData.fileBase);
      } else {
        let requestData = "";
        if (itemData !== null) {
          requestData = "documentId=" + itemData.documentId + "&b64=true";
        }
        this.GET(
          "VIEW_DOCUMENT",
          requestData,
          (res) => {
            if (res.data.statusCode === 200) {
              this.viewDocUsingBase(res.data.data);
            }
          },
          (error) => {
            console.log("Error", error);
          }
        );
      }
    },

    deleteDocument(index) {
      this.tableDocData[this.repoIndex].splice(index, 1);
      this.all_attachments = [];
    },

    browseRepo(index) {
      this.repoIndex = index;
      this.repo_dialog = true;
      
    },

    onSubmitRemarks(remarks) {
      this.updateAllDocumentsStatus(remarks);
      const request = {
        claimNumber: this.claimNumber,
        featureNumber: this.featureNumber,
        allDocumentsReceived: this.allDocumentsReceived,
        remarks: remarks,
      };

      this.POST(
        "allDocumentStatus",
        request,
        (res) => {
          if (res.data.statusCode) {
            this.getDocs();
            this.showAlert(res.data.msg);
            this.showRemarksDialog = false;
          }
        },
        (error) => {
          this.hideProgress();
          if (error.response.data.msg) {
            this.showAlert(error.response.data.msg);
          } else if (error.response.data.message.msg) {
            this.showAlert(error.response.data.message.msg);
          } else if (error.response.data.message) {
            this.showAlert(error.response.data.message);
          } else {
            this.showAlert("Something went wrong");
          }
        },
        false,
        true,
        {
          claimnumber: this.storeValues.claimNumber,
          featurenumber: this.storeValues.featureNumber,
          policynumber: this.storeValues.policyNumber,
        }
      );
    },
    uploadFromRepo() {
      this.tableDocData[this.repoIndex].push(...this.selectedRepoFiles);
      this.repo_dialog = false;
      this.selectedRepoFiles = [];
    },

    save(item) {
      let req = {
        claimId: item.main.claimId,
        claimNumber: item.main.claimNumber,
        featureNumber: item.main.featureNumber,
        documentName: item.main.documentName,
        isMandatory: item.main.isMandatory,
        documentRequested: item.main.documentRequested,
        dateRequested: item.main.dateRequested,
        dateRecieved: item.main.dateRecieved,
        documentStatus: item.main.documentStatus,
        waiverReason: item.main.waiverReason,
        isSubmitted: item.main.isSubmitted,
        document:
          this.tableDocData[item.index].length == 0
            ? item.main.document
            : this.tableDocData[item.index],
        remarks: item.main.remarks,
      };
      if (item.main.isMandatory) {
        if (
          this.tableDocData[item.index].length == 0 &&
          item.main.document.length == 0 &&
          item.main.documentStatus == "Waived"
        ) {
          if (item.main.waiverReason !== "") {
            this.showConfirm(
              "Confirmation",
              "Are you sure you want to save?",
              "Yes",
              "No",
              (Yes) => {
                this.SendReq(req);
              },
              (No) => {}
            );
          } else {
            this.showAlert("Please enter waive reason");
          }
        } else if (
          this.tableDocData[item.index].length == 0 &&
          item.main.document.length == 0 &&
          item.main.documentStatus !== "Waived"
        ) {
          this.showAlert("Please upload documents");
        } else {
          this.showConfirm(
            "Confirmation",
            "Are you sure you want to save?",
            "Yes",
            "No",
            (Yes) => {
              this.SendReq(req);
            },
            (No) => {}
          );
        }
      } else {
        if (
          item.main.documentStatus == "Waived" &&
          item.main.waiverReason == ""
        ) {
          this.showAlert("Please enter waive reason");
        } else {
          this.showConfirm(
            "Confirmation",
            "Are you sure you want to save?",
            "Yes",
            "No",
            (Yes) => {
              this.SendReq(req);
            },
            (No) => {}
          );
        }
      }
      console.log("--save req", req);
    },

    SendReq(req) {
      this.showProgress("Saving... Please wait");
      this.POST(
        "saveDocuments",
        req,
        (res) => {
          this.hideProgress();
          if (res.data.statusCode == 200) {
            this.showAlert(res.data.msg);
            this.getDocs();
          }
        },
        (error) => {
          this.hideProgress();
          if (error.response.data.msg) {
            this.showAlert(error.response.data.msg);
          } else if (error.response.data.message.msg) {
            this.showAlert(error.response.data.message.msg);
          } else if (error.response.data.message) {
            this.showAlert(error.response.data.message);
          } else {
            this.showAlert("Something went wrong");
          }
        },
        false,
        this.phaseFlag,
        {
          claimnumber: this.storeValues.claimNumber,
          featurenumber: this.storeValues.featureNumber,
          policynumber: this.storeValues.policyNumber,
        }
      );
    },

    getDocs() {
      let requestData =
        "claimNumber=" +
        this.claimNumber +
        "&featureNumber=" +
        this.featureNumber;
      this.GET(
        "getuploadocs",
        requestData,
        (res) => {
          if (
            res.data.statusCode == 200 &&
            res.data.data &&
            res.data.data.length != 0
          ) {
            this.tableDocData = [];
            this.Documents = res.data.data.data;
            this.Documents.map((ele, i) => {
              ele.docShow = { upload: false, view: false, history: false };
              ele.index = i;
              ele.total_uploadedList = [];
              let arr = [];
              this.tableDocData.push(arr);
              this.checkReqDate(ele.main.dateRequested);
            });
            this.totalListCount = res.data.data.total;
            this.allDocumentsReceived = res.data.data.allDocumentsReceived;
          } else {
            this.Documents = [];
            this.showAlert(res.data.msg);
          }
        },
        (error) => {
          this.hideProgress();
          this.Documents = [];
          if (error.response.data.msg) {
            this.showAlert(error.response.data.msg);
          } else if (error.response.data.message.msg) {
            this.showAlert(error.response.data.message.msg);
          } else if (error.response.data.message) {
            this.showAlert(error.response.data.message);
          } else {
            this.showAlert("Something went wrong");
          }
        },
        false,
        "",
        {
          claimnumber: this.storeValues.claimNumber,
          featurenumber: this.storeValues.featureNumber,
          policynumber: this.storeValues.policyNumber,
        }
      );
    },
  },
  components: { 
    HeaderOptions, 
    RemarksDialog, 
    UploadRepositoryDocumentsDialog,
  },
};
</script>

<style scoped>
.checkmargin {
  margin-top: -7px !important;
}
.dragbackground {
  background-color: #e7e7e7;
}

.closediv {
  margin-right: 42% !important;
  font-weight: 600;
  cursor: pointer;
  color: #23b1a9;
}
.mrgncls {
  margin-bottom: 15% !important;
}

.rowMargin {
  margin-left: 67%;
}

.activeIndex {
  border-bottom: 2px solid #23b1a9 !important;
  font-size: 12px;
  background-color: white !important;
  letter-spacing: 0px;
  text-transform: capitalize;
  height: 33px !important;
  border-radius: 0 !important;
  color: #23b1a9;
  font-weight: bold;
}
.uploadbtn {
  color: #fff;
}

.workitemcheck {
  height: 33px;
  margin-top: 0px;
}

.attch_width {
  width: 75%;
}

.menutext {
  font-size: 12px;
  border: 0 !important;
  background-color: white !important;
  letter-spacing: 0px;
  text-transform: capitalize;
  height: 33px !important;
  border-radius: 0 !important;
  color: #23b1a9;
  font-weight: bold;
}

.borderline {
  border-bottom: 1px solid #efefef;
}

.descrptionSpan {
  width: 85%;
  margin-top: 21px;
  font-weight: 500;
  font-size: 17px;
  letter-spacing: 0.4px;
}

.colorbg {
  background-color: #23b1a9;
  color: #fff;
}

.graybackground {
  height: 45px;
  font-size: 12px;
}

.btnstyle {
  height: 30px !important;
  text-transform: capitalize;
  font-size: 11px !important;
  letter-spacing: 0.5px;
}
.fieldWidth {
  width: 160px;
}

.tablewidth {
  max-width: 57%;
}

.doctablewidth {
  max-width: 80%;
}

.docborder {
  border: 0.5px solid #d7d7d7;
}

.dlt_text {
  color: rgb(145, 145, 145);
}

.margincls {
  margin-left: -12px;
}

.iconfont {
  font-size: 20px;
}

.uploadbr {
  height: 150px;
  max-width: 340px;
  border: 3px dashed #ccc;
  padding-top: 40px;
  cursor: pointer;
}

.textcolr {
  color: #1a0dab !important;
  font-size: 13px !important;
  cursor: pointer;
}

.iconsize {
  font-size: 17px;
}

.savebtn {
  border: 1px solid #23b1a9;
  height: 28px !important;
  font-size: 13px;
  letter-spacing: 0.5px;
  color: #23b1a9;
  text-transform: capitalize;
}
.btncommonstyle {
  text-transform: capitalize;
  height: 31px !important;
  font-size: 11px !important;
  border-radius: 4px;
}
.cancelbtn {
  border: 1px solid #4c4c4c;
}
</style>
