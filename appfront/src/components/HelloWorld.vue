
<template>
  <div>
    <el-row display="margin-top: 10px">
      <el-input v-model="inputUrl" placeholder="请输入分享地址" style="display:inline-table; width: 30%;"></el-input>
      <el-select v-model="selectedType" @change="selectType(selectedType)" size="large" style="margin: 2px; width: 10%">
        <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
            :disabled="item.disabled">
<!--          <span style="font-size: large">{{ item.label }}</span>-->
        </el-option>
      </el-select>
      <el-popover
        placement="top-start"
        width="200"
        trigger="hover">
        <span style="color: #b0b0b0">直接下载</span>
        <el-button slot="reference" :loading="loading" @click="download()" type="primary" plain style="margin: 2px;">下载</el-button>
      </el-popover>

      <el-popover
        placement="top-start"
        width="200"
        trigger="hover">
        <span style="color: #b0b0b0">获取下载链接</span>
        <el-button slot="reference" :loading="loading" @click="fetch()" style="margin: 2px;">分析</el-button>
      </el-popover>
    </el-row>

    <div v-if="downloadAddr !== ''" style="margin-top: 10px; padding-left: 20%; padding-right: 20%;">
      <div style="margin-bottom: 10px">下载地址: </div>
      <el-link :href="downloadAddr" style="font-size: large" target="_blank">
        {{ downloadAddr }}
      </el-link>
    </div>
  </div>
</template>


<script>
  function download(url, filename = '')  {
    const link = document.createElement('a');
    link.style.display = 'none';
    link.href = url;
    link.setAttribute('download', filename);

    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  export default {
    name: 'home',
    delimiters: ['[[', ']]'],
    data () {
      return {
        loading: false,
        inputUrl: '',
        selectedType: '',
        options: [],
        downloadAddr: '',
      }
  },
  mounted: function() {
      this.getTypes()
  },
  methods: {
    getTypes() {
      this.$axios.get('video/list')
        .then((res) => {
          if (res.status !== 200) {
            this.$message.error('数据读取失败!');
            return
          }
          this.options = res.data;
          this.selectedType = res.data[0].value;
        }).catch(err => this.$message.error('数据读取失败!' + this.getErrData(err)))
    },
    selectType(value) {
      this.selectedType = value
    },
    fetch() {
      let url = this.inputUrl.trim();
      if (url === '') {
        this.$message({
          message: '分享地址不能为空',
          type: 'warning'
        });
        return
      }
      this.loading = true;
      this.$axios.get('video/fetch?type=' + this.selectedType + '&url=' + url)
        .then((res) => {
          this.loading = false;
          if (res.status === 200) {
            this.downloadAddr = res.data
          } else {
            this.$message.error('地址分析失败, ' + res.data);
            this.downloadAddr = '';
          }
        }).catch((err) => {
            this.loading = false;
            this.$message.error('地址分析失败, ' + this.getErrData(err));
            this.downloadAddr = '';
        })
    },
    download() {
      let url = this.inputUrl.trim();
      if (url === '') {
        this.$message({
          message: '分享地址不能为空',
          type: 'warning'
        });
        return
      }
      this.loading = true;
      this.$axios.get('video/download?type=' + this.selectedType + '&url=' + url, {
        responseType: 'blob'
      }).then((res) => {
        // console.log(res);
        this.loading = false;
        let url;
        let filename = res.headers['content-disposition'];
        if (filename !== undefined) {
          const index = filename.indexOf('filename=');
          filename = filename.substring(index + 10, filename.length - 1);
          url = window.URL.createObjectURL(new Blob([res.data]));
          download(url, filename);
          window.URL.revokeObjectURL(url);
          return
        }

        const data = this.blobToString(res.data);
        const info = JSON.parse(data);
        // window.open(info['url'], '_blank')
        fetch(info['url'], {
          // headers: new Headers({
          //   Origin: location.origin,
          // }),
          mode: 'cors',
        })
        .then(res => res.blob())
        .then(blob => {
          url = window.URL.createObjectURL(blob);
          download(url, info['name']);
          window.URL.revokeObjectURL(url)
        });
      }).catch(err => {
        this.loading = false;
        this.$message.error('视频下载失败, ' + this.getErrData(err))
      })
    }
  }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
