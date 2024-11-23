<template>
  <div class="home">
    <div class="max-w-[1280px] px-5 xl:px-10 mx-auto py-10">
      <div v-if="hasFile" class="grid grid-cols-1 lg:grid-cols-2 gap-24">
        <div class="text-center">
          <div class="">
            <div class="font-bold text-[18px]">Ảnh đã xử lý</div>
            <div class="mt-5">
              <img :src="path1" alt="" class="mx-auto w-[500px]" />
              <div class="mt-8 flex items-center gap-5 justify-center">
                <img
                  @click="swapPath(2)"
                  :src="path2"
                  alt=""
                  class="max-w-[80px] border-2 border-black cursor-pointer"
                />
                <img
                  :src="path3"
                  @click="swapPath(3)"
                  alt=""
                  class="max-w-[80px]"
                />
              </div>
            </div>
            <input
              type="file"
              @change="handleFileChange"
              class="hidden"
              ref="fileInput"
            />
            <button
              @click="triggerFileInput"
              class="mt-8 bg-[#7F56D9] text-white px-7 py-3 rounded-lg"
            >
              Tải file
            </button>
          </div>
        </div>
        <div class="">
          <div class="">
            <span class="font-bold">1.Preprocess</span>
            <span class="block mt-2">Đạt ✅</span>
          </div>
          <div class="mt-10">
            <span class="font-bold">2. Detect text</span>
            <span class="block mt-2">Đã detect ✅</span>
          </div>
          <div class="mt-10">
            <span class="font-bold">3. Recognize character</span>
            <span class="block mt-2"
              >SNO040201021680<br />
              TƯỜNG ĐĂNG VƯƠNG QUỐC <br />
              09/01/2001 <br />
              Nam <br />
              Việt Nam <br />
              Hưng Tân, Hưng Nguyên, Nghệ An <br />
              09/01/2026 <br />
              Khối Quang Trung, Thị trấn Nam Đàn, Nam Đàn, Nghệ An
            </span>
          </div>
          <div class="mt-10">
            <span class="font-bold">4. Classify document</span>
            <span class="block mt-2">CCCD/CMND 12 số mặt trước</span>
          </div>
          <div class="mt-10">
            <span class="font-bold">5. JSON Output </span>
            <span class="block mt-2"
              >[ { “type”: “CCCD/CMND 12 số mặt trước” "idNumber":
              "SNO040201021680", "fullName": "TƯỜNG ĐĂNG VƯƠNG QUỐC",
              "dateOfBirth": "09/01/2001", "sex": "Nam", "national": "Việt Nam",
              "placeOfOrigin": "Hưng Tân, Hưng Nguyên, Nghệ An", "dateOfExpiry":
              "09/01/2026", "placeOfResidence": "Khối Quang Trung, Thị trấn Nam
              Đàn, Nam Đàn, Nghệ An" } ]
            </span>
          </div>
          <div class="mt-10">
            <span class="font-bold">6. Correct character</span>
            <span class="block mt-2"
              >[ { “type”: “CCCD/CMND 12 số mặt trước” "idNumber":
              "SNO040201021680", "fullName": "TƯỜNG ĐĂNG VƯƠNG QUỐC",
              "dateOfBirth": "09/01/2001", "sex": "Nam", "national": "Việt Nam",
              "placeOfOrigin": "Hưng Tân, Hưng Nguyên, Nghệ An", "dateOfExpiry":
              "09/01/2026", "placeOfResidence": "Khối Quang Trung, Thị trấn Nam
              Đàn, Nam Đàn, Nghệ An" } ]
            </span>
          </div>
        </div>
      </div>
      <div v-if="!hasFile" class="mx-auto mt-24 text-center">
        <div class="font-bold text-[20px]">Hệ thống trích xuất dữ liệu OCR</div>
        <span class="block mt-6">Bạn chưa có file nào</span>
        <input
          type="file"
          @change="handleFileChange"
          class="hidden"
          ref="fileInput"
        />
        <button
          @click="triggerFileInput"
          class="mt-5 bg-[#7F56D9] text-white px-7 py-3 rounded-lg"
        >
          Tải file
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import axios from "axios";
export default {
  name: "HomeView",
  setup() {
    const hasFile = ref(false);
    const file = ref(null);
    const path1 = ref("");
    const path2 = ref("");
    const path3 = ref("");

    const triggerFileInput = () => {
      const fileInput = document.querySelector("input[type='file']");
      fileInput.click();
    };

    const handleFileChange = (event) => {
      const selectedFile = event.target.files[0]; // Lấy file đầu tiên
      if (selectedFile) {
        file.value = selectedFile;
        console.log(path1.value);
        uploadFile(selectedFile); // Gửi file lên server
      }
    };
    const uploadFile = async (file) => {
      try {
        // Tạo FormData và thêm file
        const formData = new FormData();
        formData.append("file", file);

        // Gửi yêu cầu POST để upload file
        const response = await axios.post(
          "http://127.0.0.1:5000/upload",
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        console.log("File đã được gửi thành công:", response);

        // Gắn giá trị path từ response
        path1.value = response.data.path1;
        path2.value = response.data.path2;
        path3.value = response.data.path3;
        hasFile.value = true; // Đã có file

        // Gửi yêu cầu POST với path1
        const imageUrl = path1.value;
        const postResponse = await axios.post(
          "http://127.0.0.1:3000/upload",
          {
            imageUrl: imageUrl,
          },
          {
            headers: {
              "Content-Type": "application/json",
            },
          }
        );
        console.log(postResponse);

        console.log("Gửi dữ liệu path1 thành công:", postResponse);
      } catch (error) {
        console.error("Đã xảy ra lỗi:", error);
      }
    };

    const swapPath = (tmp) => {
      if (tmp === 2) {
        const pathTmp = path1.value;
        path1.value = path2.value;
        path2.value = pathTmp;
      } else if (tmp === 3) {
        const pathTmp = path1.value;
        path1.value = path3.value;
        path3.value = pathTmp;
      }
    };

    return {
      hasFile,
      path1,
      path2,
      path3,
      swapPath,
      triggerFileInput,
      handleFileChange,
    };
  },
};
</script>
