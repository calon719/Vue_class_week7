<template>
  <div class="modal fade" id="productModal" tabindex="-1"
    aria-labelledby="productModalLabel" aria-hidden="true"
    ref="modal">
    <div class="modal-dialog modal-xl">
      <div class="modal-content">
        <div class="modal-header bg-dark text-white">
          <h3 class="modal-title" id="productModalLabel">
            {{ status === 'add' ? '新增' : '編輯'}}產品資料
          </h3>
          <button type="button" class="btn-close btn-close-white"
            data-bs-dismiss="modal" aria-label="Close"></button>
        </div>

        <Form ref="form" v-slot="{ errors, meta }" @submit="updateProduct">
          <div class="modal-body">
            <div class="row">
              <div class="col-md-4 mb-3 mb-md-0">
                <div class="mb-3">
                  <small class="text-danger">※ 圖片大小不可超過 3 MB</small>
                  <div class="mb-2">
                    <label class="form-label" for="imgFile">產品主要圖片</label>
                    <div class="btn-group w-100 mb-1" role="group"
                      aria-label="Upload image button group">
                      <button type="button" class="btn btn-outline-secondary"
                        @click="imgBtn.main = 'upload'">上傳圖片</button>
                      <button type="button" class="btn btn-outline-secondary"
                        @click="imgBtn.main = 'url'">輸入網址</button>
                    </div>
                    <div v-show="imgBtn.main === 'upload'">
                      <input type="file"
                        v-if="!isUploading"
                        class="form-control" id="imgFile"
                        placeholder="請選擇要上傳的圖片檔案"
                        :value="resetStr"
                        @change="uploadImg($event, 'mainImg')" />
                      <div v-else
                        class="d-flex justify-content-center opacity-50 py-2">
                        <div class="spinner-border text-dark"
                          role="status">
                          <span class="visually-hidden">Loading...</span>
                        </div>
                      </div>
                    </div>

                    <input type="text" class="form-control"
                      placeholder="請輸入圖片網址" aria-label="輸入圖片網址"
                      aria-describedby="button-addon2"
                      v-show="imgBtn.main === 'url'"
                      v-model="item.imageUrl" />
                  </div>
                  <div v-show="item.imageUrl">
                    <input type="text"
                      class="form-control mb-1" aria-label="欲上傳產品圖片"
                      :value="item.imageUrl" readonly />
                    <img class="img-fluid" :src="item.imageUrl" alt="產品主圖片">
                  </div>
                </div>

                <div class="mb-2">
                  <label class="form-label" for="otherImgFile">產品其他圖片</label>
                  <div class="btn-group w-100 mb-1" role="group"
                    aria-label="Upload image button group">
                    <button type="button" class="btn btn-outline-secondary"
                      @click="imgBtn.other = 'upload'">上傳圖片</button>
                    <button type="button" class="btn btn-outline-secondary"
                      @click="imgBtn.other = 'url'">輸入網址</button>
                  </div>
                  <div v-show="imgBtn.other === 'upload'">
                    <input type="file"
                      class="form-control" id="otherImgFile"
                      placeholder="請選擇要上傳的圖片檔案"
                      v-if="!isUploading"
                      :value="resetStr"
                      @change="uploadImg($event, 'otherImg')" />
                    <div v-else
                      class="d-flex justify-content-center opacity-50 py-2">
                      <div class="spinner-border text-dark"
                        role="status">
                        <span class="visually-hidden">Loading...</span>
                      </div>
                    </div>
                  </div>
                  <div class="input-group mb-3"
                    v-if="imgBtn.other === 'url'">
                    <input type="text" class="form-control"
                      placeholder="請輸入圖片網址"
                      aria-label="輸入圖片網址"
                      aria-describedby="otherImgBtn"
                      v-model="imgUrl" />
                    <button type="button"
                      class="btn btn-outline-secondary" id="otherImgBtn"
                      @click="pushImg">新增</button>
                  </div>
                </div>
                <div v-show="item.imagesUrl?.length">
                  <div class="input-group mb-1"
                    v-for="(image, i) in item.imagesUrl" :key="image">
                    <input type="text" class="form-control"
                      aria-label="欲上傳產品圖片" aria-describedby="button-deleteImgUrl"
                      :value="image" readonly />
                    <button type="button"
                      class="btn btn-danger"
                      id="button-deleteImgUrl"
                      @click="item.imagesUrl.splice(i, 1)">刪除</button>
                  </div>
                </div>
              </div>

              <div class="col-md-8">
                <div class="row">
                  <div class="mb-3">
                    <label class="form-label" for="title">
                      產品標題
                      <span class="text-danger">*</span>
                    </label>
                    <Field id="title" class="form-control" type="text"
                      name="標題" placeholder="請輸入產品標題"
                      rules="required" v-model="item.title"
                      :class="{ 'is-invalid': errors['標題'] }" />
                    <ErrorMessage name="標題" class="invalid-feedback"></ErrorMessage>
                  </div>
                  <div class="col-md-6 mb-3">
                    <label class="form-label" for="category">
                      分類
                      <span class="text-danger">*</span>
                    </label>
                    <Field id="category" class="form-control" type="text"
                      name="分類"
                      placeholder="請輸入產品分類"
                      rules="required"
                      :class="{ 'is-invalid': errors['分類'] }"
                      v-model="item.category" />
                    <ErrorMessage name="分類" class="invalid-feedback"></ErrorMessage>
                  </div>
                  <div class="col-md-6 mb-3">
                    <label class="form-label" for="unit">
                      單位
                      <span class="text-danger">*</span>
                    </label>
                    <Field id="unit" class="form-control" type="text"
                      placeholder="請輸入產品的單位"
                      name="單位"
                      rules="required"
                      :class="{ 'is-invalid': errors['單位'] }"
                      v-model="item.unit" />
                    <ErrorMessage name="單位" class="invalid-feedback"></ErrorMessage>
                  </div>
                  <div class="col-md-6 mb-3">
                    <label class="form-label" for="originPrice">
                      原價
                      <span class="text-danger">*</span>
                    </label>
                    <Field id="originPrice" class="form-control"
                      type="number" min="0" placeholder="請輸入產品原價"
                      name="原價"
                      rules="required|min_value:0"
                      :class="{ 'is-invalid': errors['原價'] }"
                      v-model.number="item.origin_price" />
                    <ErrorMessage name="原價" class="invalid-feedback"></ErrorMessage>
                  </div>
                  <div class="col-md-6 mb-3">
                    <label class="form-label" for="price">
                      售價
                      <span class="text-danger">*</span>
                    </label>
                    <Field id="price" class="form-control"
                      type="number" min="0" placeholder="請輸入產品售價"
                      name="售價"
                      rules="required|min_value:0"
                      :class="{ 'is-invalid': errors['售價'] }"
                      v-model.number="item.price" />
                    <ErrorMessage name="售價" class="invalid-feedback"></ErrorMessage>
                  </div>
                </div>
                <hr>
                <div class="mb-3">
                  <label class="form-label" for="description">產品描述</label>
                  <textarea id="description" class="form-control"
                    type="text" row="100" placeholder="請輸入產品描述"
                    v-model="item.description"></textarea>
                </div>
                <div class="mb-3">
                  <label class="form-label" for="content">內容說明</label>
                  <textarea id="content" class="form-control"
                    type="text" row="100" placeholder="請輸入產品內容說明"
                    v-model="item.content"></textarea>
                </div>

                <div class="mb-3">
                  <label class="form-label" for="productTag">產品標籤</label>
                  <select class="form-select" id="productTag"
                    aria-label="Product Tag"
                    v-model="item.tag">
                    <option disabled>請選擇產品標籤</option>
                    <option value="0">無</option>
                    <option value="1">人氣精選</option>
                    <option value="2">新品上市</option>
                  </select>
                </div>
                <div class="d-flex justify-content-end">
                  <div class="form-check form-switch">
                    <input type="checkbox" role="switch"
                      id="is_enabled" class="form-check-input"
                      :true-value="1" :false-value="0"
                      v-model="item.is_enabled" />
                    <label class="form-check-label" for="is_enabled">是否啟用</label>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary px-4"
              data-bs-dismiss="modal">取消</button>
            <button type="submit" class="btn btn-primary px-4"
              :disabled="isBtnLoading || !meta.valid">
              <span v-if="!isBtnLoading">確認</span>
              <span v-else class="spinner-border spinner-border-sm"
                role="status" aria-hidden="true"></span>
            </button>
          </div>
        </Form>
      </div>
    </div>
  </div>
</template>

<script>
import modalMixin from '@/mixins/modalMixin';

export default {
  data() {
    return {
      modal: {},
      item: this.product,
      imgBtn: {
        main: '',
        other: '',
      },
      imgUrl: '',
      resetStr: '',
      isUploading: false,
      isBtnLoading: false,
    };
  },
  props: ['product', 'status'],
  watch: {
    product() {
      this.item = this.product;
    },
  },
  mixins: [modalMixin],
  inject: ['pushToastMessage'],
  methods: {
    uploadImg(e, status) {
      const imgFile = e.target.files[0];
      const formData = new FormData();
      formData.append('file-to-upload', imgFile);

      const api = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/upload`;
      this.isUploading = true;
      this.$http.post(api, formData)
        .then((res) => {
          this.pushToastMessage(res.data.success, '圖片上傳');
          const imgUrl = res.data.imageUrl;

          switch (status) {
            case 'mainImg':
              this.item.imageUrl = imgUrl;
              break;
            case 'otherImg':
              if (!this.item.imagesUrl) {
                this.item.imagesUrl = [];
                this.item.imagesUrl.push(imgUrl);
              } else {
                this.item.imagesUrl.push(imgUrl);
              }
              break;
            default:
              break;
          }
        }).catch((err) => {
          console.dir(err);
        }).then(() => {
          this.isUploading = false;
          this.resetStr = '';
        });
    },
    pushImg() {
      if (this.imgUrl === '') {
        return; // eslint-disable-line
      }

      if (!this.item.imagesUrl) {
        this.item.imagesUrl = [];
        this.item.imagesUrl.push(this.imgUrl);
      } else {
        this.item.imagesUrl.push(this.imgUrl);
      }
    },
    updateProduct() {
      let method = '';
      let url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/product`;

      switch (this.status) {
        case 'add':
          method = 'post';
          break;
        case 'edit':
          method = 'put';
          url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/product/${this.item.id}`;
          break;
        default:
          break;
      }

      this.isBtnLoading = true;
      this.$http[method](url, { data: this.item })
        .then((res) => {
          this.modal.hide();
          this.pushToastMessage(res.data.success, '產品更新');
          this.$emit('update', true);
        }).catch((err) => {
          console.dir(err);
          this.pushToastMessage(err.response.data.success, '產品更新');
        }).then(() => {
          this.isBtnLoading = false;
        });
    },
  },
};
</script>
