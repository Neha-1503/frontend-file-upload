<template>
  <div class="frame">
  <div class="center">
			<div class="upload-header">Upload File </div>
			<div class="upload-body">
				<div class="dropzone">
					<div class="content">
						<label for="fileUpload">
              <i v-if="!file" class="bi bi-cloud-arrow-up"></i>
              <i v-else class="bi bi-file-earmark-plus"></i>
            </label>
						<input type="file" id="fileUpload" class="input" @change="fileSelected">
					</div>
				</div>
        <input type="text" v-model="fileName" class="form-control w-70 file-name" id="fileName" placeholder="Enter File Name*" @keypress="validateText">
        <div v-if="!isValidFileType" class="error-banner">Only JPG, PDF or DOCX files allowed.</div>
				<button class="upload-btn" :disabled="disableUpload" @click="uploadFile"> Upload file</button>
			</div>
  </div>
</div>

<div class="d-flex align-items-center justify-content-center" v-if="files?.length">
  <table class="table table-striped w-70" :key="files">
    <thead>
      <tr class="table-primary">
        <th scope="col">File Name</th>
        <th scope="col">Date Uploaded</th>
        <th class="text-center">Download</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="file of files" :key="file">
        <td>{{ file.fileName }}</td>
        <td>{{formatUploadDate(file.dateUploaded)}}</td>
        <td class="text-center">
          <button type="button" class="btn btn-outline-primary" @click="downloadFile(file.file_id)">
            <i class="bi bi-cloud-arrow-down"></i>
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</div>

</template>

<script>
import { ref, computed, onBeforeMount } from 'vue';
import axios from 'axios';

export default {
  setup() {
    onBeforeMount(async () => {
      await getFiles();
    });

    let file = ref();
    const files = ref();
    const fileName = ref();

    const fileSelected = (selectedFile) => {
      file.value = selectedFile.target.files[0];
    }

    const disableUpload = computed(() => {
      if(!(file.value && fileName.value)) return true;
      if (!isValidFileType.value) return true;
      return false;
    });

    const isValidFileType = computed(() => {
      if (!file.value) return true;
      return (file.value && (file.value?.type === 'application/pdf' || file.value?.type === "application/vnd.openxmlformats-officedocument.wordprocessingml.document" || file.value?.type === "image/jpeg"))
    })

    const uploadFile = async () => {
      const formData = new FormData();
      formData.append('file', file.value, fileName.value);
      try {
        await axios({
          method: 'post',
          headers: {
            'Content-Type': 'multipart/form-data',
          },
          url: 'http://localhost:8000/fileUpload/file',
          data: formData
        })
        setTimeout(async () => {
          file.value = undefined;
          fileName.value = '';
          await getFiles();
        }, 500);
      } catch (err) {
        window.alert('We faced problem while uploading file, please try again later')
      }
    }

    const getFiles = async () => {
    try {
      await axios({
        method: 'get',
        url: 'http://localhost:8000/fileUpload/files',
      }).then((res) => {
        files.value = res.data;
        files.value.sort((file1,file2) => {
          const timeA = new Date(file1.dateUploaded);
          const timeB = new Date(file2.dateUploaded);

          return timeB - timeA;
        })
      })
    } catch (err) {
      window.alert('We faced a problem while fethcing files, please try again later')
    }
    }

    const downloadFile = async(fileId) => {
      try {
        await axios({
          method: 'get',
          url: `http://localhost:8000/fileUpload/download/${fileId}`,
          responseType: 'arraybuffer',
        }).then((res) => {
          const link = document.createElement('a');
      
          if (link.download !== undefined) {
            const blob = new Blob([res.data], {type: 'application/pdf'});
            const url = URL.createObjectURL(blob);
            link.href = url;
            link.target = '_blank';
            link.click();
          }
        })
      } catch (err) {
        window.alert('We faced problem while downloading the file, please try again later.')
      }
    }

    const formatUploadDate = (dateISOString) => {
      const date = new Date(dateISOString);
      return `${date.getDate()}/${date.getMonth()}/${date.getFullYear()} ${date.getHours()}:${date.getMinutes()}`
    }

    const validateText = (event) => {
      const char = event.key;
      const regex = /^[A-Za-z0-9 ]+$/;
      if (regex.test(char)) return true;
      event.preventDefault();
      return false;
    }


    // expose to template and other options API hooks
    return {
      validateText,
      formatUploadDate,
      downloadFile,
      uploadFile,
      fileSelected,
      isValidFileType,
      disableUpload,
      files,
      file,
      fileName
    }
  },
}

</script>

<style scoped>
@import url(https://fonts.googleapis.com/css?family=Open+Sans:700,400,300);

.frame {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 400px;
  border-radius: 2px;
  overflow: hidden;
  color: #676767;
}

.center {
  width: 70%;
  height: 260px;
  background-color: #fff;
  box-shadow: 8px 10px 15px 0 rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}


.upload-header {
  width: 100%;
  height: 50px;
  border-bottom: 1px solid #d8d8d8;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #596FB7;
  color: white;
}

.upload-body {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  width: 100%;
  height: 209px;
  padding: 5px;
}

.w-70 {
  width: 70%
}


.upload-btn {
  border: none;
  color: #fff;
  line-height: 40px;
  font-size: 14px;
  background-color: #11235A;
  width: 70%;
  height: 40px;
  border-radius: 3px;
  bottom: 24px;
  left: 80px;
  cursor: pointer;
}
.file-name {
  font-size: 12px
}
.error-banner {
  color: red;
  font-size: 12px;
}

.upload-btn:disabled {
  opacity: 0.5;
  cursor:not-allowed;
}

.dropzone {
  z-index: 1;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  border: 1px dashed #a4a4a4;
  border-radius: 3px;
  text-align: center;
}
  .content {
  width: 100px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 52px;
  color: #a4a4a4
  }

  .input {
    opacity: 0;
    display: none;
  }
</style>

