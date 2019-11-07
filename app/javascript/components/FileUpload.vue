<template>
  <div>
    <div class="row">
      <div class="form-group">
        <div class="col-md-12" style="font-size: 18px">{{ label }}</div>
      </div>
    </div>

    <div class="row">
      <div class="form-group">
        <div class="col-md-12 upload">
          <div class="form-table">
            <div class="form-cell cell-label">
              <span class="desc_url">{{ $t("order_comparer.desc_url") }}</span>
            </div>
            <div class="input-group">
              <input type="text" class="form-control" :placeholder="nameFile ? nameFile : placeholder" :class=" {active: nameFile}">
              <span class="input-group-btn">
                <button v-on:click.prevent="selectFile" variant="primary" class="btn btn-primary button-size btn-file"><i class="glyphicon glyphicon-folder-open"></i>&nbsp; {{ $t('order_comparer.import') }}</button>
                <input type="file" @change="onChange($event)" :accept="typeFile" :id="name" style="display: none">
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    props: {
      updateState: {
        type: Function
      },
      label: {
        type: String
      },
      name: {
        type: String
      },
      placeholder: {
        type: String
      },
      typeFile: {
        type: String
      },
      fileSelect: {
        type: Object
      }
    },
    data: function() {
      return {
        nameFile: null
      }
    },

    methods: {
      selectFile: function() {
        $(`#${this.name}`).click();
      },

      onChange: function(event) {
        const file = event.target.files[0];
        if(file) {
          this.updateState(this.fileSelect, file);
          this.nameFile = file.name
        }
      },



      // readFileExcel(ev) {
      //   const file = ev.target.files[0];
      //   const reader = new FileReader();
      //   reader.onload = e => this.updateState("fileExcel", e.target.result);
      //   this.updateState("nameExcel", file.name);
      //   reader.readAsBinaryString(file);

      // },
      
      // readFileGoogle(ev) {
      //   const file = ev.target.files[0];
      //   const reader = new FileReader();
      //   reader.onload = e => this.updateState("fileGoogle", e.target.result);
      //   this.updateState("nameGoogle", file.name);
      //   reader.readAsText(file, 'SHIFT_JIS');
      // },

      // readFileYahoo(ev) {
      //   const file = ev.target.files[0];
      //   const reader = new FileReader();
      //   reader.onload = e => this.updateState("fileYahoo", e.target.result);
      //   this.updateState("nameYahoo", file.name);
      //   reader.readAsText(file, 'SHIFT_JIS');
      // }
    }
  }
</script>
<style>
.col-md-12.upload {
  padding-top: 7px;
  padding-bottom: 5px;
}

span.desc_url {
  font-size: 14px;
}

input.form-control.active::placeholder {
  color: #333333;
}
</style>