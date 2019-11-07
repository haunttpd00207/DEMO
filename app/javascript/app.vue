<template>
  <div>
    <h4 class="page-title custom-title">{{ $t("order_comparer.title") }}</h4>
    <div class="content-app">
      <div class="x_panel">
        <div class="x_content">
          <FileUpload
            :updateState="updateState"
            :placeholder="$t('order_comparer.placeholder_fmt')"
            :label="$t('order_comparer.label_fmt')"
            :fileSelect="fmt_file"
            name="file-fmt"
            typeFile=".xlsm, .xlsx"
          />
          
          <FileUpload
            :updateState="updateState"
            :placeholder="$t('order_comparer.placeholder_google')"
            :label="$t('order_comparer.label_google')"
            :fileSelect="google_csv_file"
            name="file-google"
            typeFile=".csv"
          />

          <FileUpload
            :updateState="updateState"
            :placeholder="$t('order_comparer.placeholder_yahoo')"
            :label="$t('order_comparer.label_yahoo')"
            :fileSelect="yahoo_csv_file"
            name="file-yahoo"
            typeFile=".csv"
          />
          <DownloadButton
            :submit="submit"
          />
        </div>
      </div>
    </div>
    <div class="x_panel no-border no-background" >
      <div class="row">
        <div class="col-md-12">
          <div class="form-table">
            <div class="form-row">
              <div class="form-cell cell-label">
                <p v-html="$t('order_comparer.note_html')"></p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  import bootbox from 'bootbox'
  import _ from 'lodash'
  import order from './api/order_comparers'
  import FileUpload from './components/FileUpload.vue'
  import DownloadButton from './components/DownloadButton.vue'
  import XlsxPopulate from 'xlsx-populate'
  export default {
    data() {
      return {
        fmt_file: {
          file: null,
          uuid: null,
          formData: null
        },
        google_csv_file: {
          file: null,
          uuid: null,
          formData: null
        },
        yahoo_csv_file: {
          file: null,
          uuid: null,
          formData: null
        },
        validTypeExcel: {
          search: "公式依頼FMT_サーチ",
          video: "公式依頼FMT_ディスプレイ",
          display: "公式依頼FMT_動画" 
        },
        validTypeYahoo: {
          search: "マッチタイプ",
          video: "ターゲティングの種類",
          display: "ターゲティングの種類" 
        }
      }
    },

    components: {
      FileUpload,
      DownloadButton
    },

    methods: {
      submit: function(e) {
        bootbox.hideAll();
        var self = this;
        var errors = []
        if (!self.fmt_file.file) {
          errors.push(self.$t("error.error_missing_excel"));
        }
        if (!self.google_csv_file.file && !self.yahoo_csv_file.file) {
          errors.push(self.$t("error.error_missing_google_yahoo"));
        }
        if (errors.length > 0) {
          bootbox.alert({message: errors.join("<br>"), className: "modal-scrollable"});
        } else {
          bootbox.confirm({
            message: self.$t("error.sure") ,
              buttons: {
                confirm: {
                  label: self.$t("order_comparer.yes"),
                  className: 'btn-primary margin-right'
                },
                cancel: {
                  label: self.$t("order_comparer.no"),
                  className: 'btn-default pull-right'
                }
              },
            callback: function (result) {
              self.checkFmtFile(self.checkCsvFile())
            }
          });
        }
      },

      checkFmtFile: function(successCallBack) {
        var self = this;
        setTimeout(function() {
          XlsxPopulate.fromDataAsync(self.fmt_file.file).then(function(wb) {
            var errors = []
            var currentSheet = wb.sheet("表紙");
            if (currentSheet) {
              var coverName = currentSheet.cell("B14").value();
              if (!_.startsWith(coverName, self.validTypeExcel)) {
                errors.push(self.$t("error.not_find"));
              } else {
                successCallBack()
                debugger
              }
            }
          }, function(error) { 

          });
        }, 1000);
      },
      checkCsvFile(successCallBack) {
          var self = this;
          var reader = new FileReader();
          var blob = this.yahoo_csv_file.file.slice(0, 5000);
          var retryRead = false;
          reader.onloadend = function(e) {
            var headers = this.result.split('\n').shift().split(',');
            if (_.difference(self.config.csv.required, headers).length !== 0) {
              debugger
              if (!retryRead) {
                retryRead = true
                reader.readAsText(blob, 'SHIFT_JIS');
              } else {
                hiddenLoader();
                bootbox.alert({message: "#{I18n.t 'errors.cr_comparer.draf_result_invalid'}"});
              }
            } else if (successCallBack) {
              successCallBack();
            }
          };
          reader.onerror = function(e) {
            bootbox.alert({message: 'Read CSV Error: ' + e.target.error.code});
            reader.abort();
          };
          reader.readAsText(blob);
        },

      // checkCsvFile(successCallBack) {
      //   var self = this;
      //   var errors = []
      //   var reader = new FileReader();
      //   var blob = self.yahoo_csv_file.file.slice(0, 5000);
      //   reader.readAsText(blob, 'SHIFT_JIS');
      //   reader.onloadend = function(e) {
      //     var headers = this.result.split('\n').shift().split(',');
      //     if (headers) {
      //       if (!_.startsWith(headers, self.validTypeYahoo)) {
      //         debugger
      //         errors.push(self.$t("error.not_match"));
      //       } else {
      //         successCallBack()
      //         debugger
      //       }
      //       debugger
      //     }
      //   };
      //   reader.onerror = function(e) {
      //     bootbox.alert({message: 'Read CSV Error: ' + e.target.error.code});
      //     reader.abort();
      //   };
      //   reader.readAsText(blob);
      // },
      
      // classifyFileYahoo: function() {
      //   var self = this;
      //   const workbook = XLSX.read(self.fileYahoo, {type:'binary'});
      //   const first_sheet_name = workbook.SheetNames[0];
      //   const sheet = workbook.Sheets[first_sheet_name];
      //   var typeOfYahoo =_.find(this.validTypeYahoo, function(type){
      //     return sheet && _.indexOf(sheet, type.parttern)
      //   })
      //   if (!typeOfYahoo) return null
      //   return typeOfYahoo.type
      // },

      // classifyFile: function() {
      //   var self = this;
      //   const workbook = XLSX.read(self.fileExcel, {type:'binary'});
      //   const worksheet = workbook.Sheets[self.sheetNameExcel];
      //   var desired_cell = worksheet[self.cellNameExcel];
      //   var desired_value = (desired_cell ? desired_cell.v : undefined);
      //   var typeOf =_.find(this.validTypes, function(type){
      //     return desired_value && _.startsWith(desired_value, type.parttern)
      //   })
      //   if (!typeOf) return null
      //   return typeOf.type
      // },
      
      updateState(type, result) {
        type.file = result
      }
    },

    computed: {
      getTypeAd() {
        var self = this
        var type_ads = '';
        if (!self.isNothing) {
          if (self.isSearch) {
            type_ads = 'サーチの場合'
          } else if (self.isVideo) {
            type_ads = 'ディスプレイの場合'
          } else if (self.isDisplay) {
            type_ads = '動画の場合'
          }
            return type_ads
        }
      }
    }
  }
</script>
<style scoped>
  .form-control:disabled,
   
  .form-control[readonly] {
    background-color: white;
  }

  span {
    font-size: 14px;
  }

  .form-cell.cell-label {
    font-size: 14px;
  }
</style>