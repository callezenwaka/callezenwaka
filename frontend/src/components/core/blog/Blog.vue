<template>
  <div>
    <BreadCrumb></BreadCrumb>
    <section class="blog" v-if="isAuthenticated && !!blog && isEditing">
      <h1 class="blog--title">Edit Blog</h1>
      <form @submit.prevent="handleUpdate">
        <div class="form--item">
          <label for="title" class="form--label">Title:</label>
          <input type="text" class="form--input" name="title" id="title" v-model="title"
             placeholder="New blog title" aria-required="true" aria-invalid="false" autocomplete="title" required>
        </div>
        <div class="form--item">
          <label for="summary" class="form--label">Summary:</label>
          <textarea 
            type="text" 
            class="form--input" 
            name="summary" 
            id="summary" 
            v-model="summary"
            placeholder="New blog summary" 
            @change="handleAdjustFocus($event)" 
            @keyup="handleAdjustFocus($event)"
            @focus="handleAdjustFocus($event)" 
            @blur="handleAdjustBlur($event)" 
            aria-required="true" 
            aria-invalid="false"
            autocomplete="summary" 
            required
          >
          </textarea>
        </div>
        <div class="form--item">
          <div style="margin-top: 0.4rem;">
            <QuillEditor class="form--editor" v-model:content="blog.content" content-type="html" toolbar="full" theme="snow" />
          </div>
        </div>
        <div class="form--item multiple">
          <div class="form--item">
            <label for="status" class="form--label required">Status: </label>
            <!-- <br> -->
            <select class="form--input" v-model="status" name="status" id="status">
              <option disabled value="">--</option>
              <!-- As per discussion here: https://stackoverflow.com/a/45265037 -->
              <option :value='false'>False</option>
              <option :value='true'>True</option>
            </select>
          </div>
          <div class="form--item">
            <label for="tags" class="form--label required">Tags:</label>
            <div class="multiselect">
              <div class="selectBox" @click="handleExpandOptions">
                <select class="form--input">
                  <option>Select an option ({{ tags?.length }})</option>
                </select>
                <div class="overSelect"></div>
              </div>
              <div class="tags" id="tags" ref="tagRef" @mouseleave="handleExpandOptions">
                <label :for="tag" v-for="(tag, i) in allTags" :key="i">
                  <input type="checkbox" :value="tag" :id="tag" v-model="tags"/>
                  <span>{{tag}}</span>
                </label>
              </div>
            </div>
          </div>
        </div>
        <div class="form--item multiple">
          <button type="submit" class="form--button" @click="handleEditing">Cancel</button>
          <button type="submit" class="form--button">Submit</button>
        </div>
      </form>
    </section>
    <section v-else class="blog">
      <Platform v-bind="{ title: blog?.title!, summary: blog?.summary! }"></Platform>
      <h1 class="blog--title">{{ blog?.title }}</h1>
      <img v-if="blog?.avatar" class="blog--avatar" :src="blog?.avatar" :alt="blog?.title" type="image/*" :title="blog?.title">
      <div class="blog--summary">
        <span v-html="blog?.content"></span>
        <!-- <p>{{ blog?.content }}</p> -->
      </div>
      <p class="blog--tags">Tags: {{ blog?.tags }}</p>
      <div v-if="isAuthenticated" class="form--item multiple">
        <button type="button" class="form--button" @click="handleEditing">Edit</button>
        <button type="button" class="form--button">Delete</button>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import BreadCrumb from "@/components/partials/BreadCrumb.vue";
import Platform from "@/components/partials/Platform.vue";
import { handleAdjustFocus, handleAdjustBlur } from "@/utils";
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css';
import { useBlogStore, useAuthStore } from '@/stores'
import { /* useRoute, */ useRouter } from 'vue-router'
import { storeToRefs } from "pinia";
import { ref, /* computed */ } from "vue";
// const props = defineProps({
//   blog: {
//     type: Object,
//     required: true,
//   },
// });
const { isAuthenticated } = storeToRefs(useAuthStore());
const { tags: allTags, blog } = storeToRefs(useBlogStore());
const blogStore = useBlogStore();
const router = useRouter();
// const route = useRoute();
// const shareUrl = ref(window.location.href);

// const { id } = route.params;
// const blog = computed(() => blogStore.getOneBlogById(id.toString()));
// const blog = computed(() => blogStore.getBlog(id.toString()));
// const options = ref(['Politics', 'Economy', 'Social', 'Art', 'Technology']);
const isEditing = ref(false);
const title = ref(blog?.value?.title)
const summary = ref(blog?.value?.summary)
const content = ref(blog?.value?.content)
const status = ref(blog?.value?.status)//: false,
const tags = ref(blog?.value?.tags);
const tagRef = ref<(HTMLElement)>();
var expanded = ref(false);

// console.log('blog: ', blog);

const handleEditing = () => {
  isEditing.value = !isEditing.value;
};

const handleExpandOptions = () => {
  // var roles = rolesRef.value;
  if (!expanded.value) {
    tagRef!.value!.style.display = "block";
    expanded.value = true;
  } else {
    tagRef!.value!.style.display = "none";
    expanded.value = false;
  }
};

const handleUpdate = async() => {
  blogStore.updateBlog({
    ...blog!.value!, 
    title: title!.value!, 
    summary: summary!.value!,
    content: content!.value!, 
    status: status!.value!, 
    tags: tags!.value!
  });
  isEditing.value = !isEditing.value;
  await router.push({ name: 'Blog', params: { id: blog!.value!.id } });
  // flash message here!
};

</script>

<style scoped lang="css">
.blog {
  /* max-width: 960px; */
  height: 375vh;
  height: 100%;
  margin: 0 auto;
  padding-top: 1rem;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
}

.blog--summary {
	text-align: justify;
	/* margin: 1.5rem 0rem; */
}

.blog--tags {
	text-align: left;
  margin-top: 2rem;
  margin-bottom: 2rem;
	/* margin: 1.5rem 0rem; */
}

.form--item {
  display: flex;
  flex-direction: column;
  width: 100%;
  border: none;
  margin: 0px 0px 1.5rem;
  padding: 0px;
}

.form--item.multiple {
  /* display: flex; */
  flex-direction: row;
  column-gap: 10px;
}

.form--label {
  color: var(--text-color-primary);
  position: relative;
  height: 16px;
  text-align: left;
  font-size: inherit;
  font-weight: bold;
  line-height: 16px;
  letter-spacing: 0.02rem;
}
.form--input {
	color: var(--text-color-primary);
  height: 3rem;
  width: 100%;
  font-size: inherit;
  line-height: 3rem;
  border-radius: 5px;
  border: 1px solid rgba(0, 0, 0, 0.5);
  outline: none;
  padding-left: 10px;
  margin-top: 5px;
}
.form--editor {
  background-color: inherit;
}
.form--buttons {
  width: 100%;
  margin-top: 16px;
  font-size: 1.2rem !important;
  line-height: 3rem;
  border-radius: 5px;
  /* border: 2px solid transparent; */
  /* background-color: transparent; */
  outline: none;
  cursor: not-allowed;
  padding-left: 10px;
  transition: all 150ms ease-in-out 0s;
  box-shadow: 0 1px 2px 0 rgb(60 64 67 / 30%), 0 1px 3px 1px rgb(60 64 67 / 15%);
}

.form--button {
	background-color: var(--button-background-color-primary);
	color: var(--button-text-color-primary);
	border: none;
	border: 0.1rem solid var(--text-color-primary);
	border-radius: 0.5rem;
	text-align: center;
	font-size: inherit;
	width: 100%;
	padding: 0.5rem 1rem;
	cursor: pointer;
	transition: all 1s ease-out;
}

.form--button.danger {
  background-color: var(--background-color-danger);
}

.form--button:hover {
	filter: drop-shadow(0 0 1.5rem var(--drop-shadow-primary));
}

/* multi select */
.multiselect {
  width: 100%;
}
.selectBox {
  position: relative;
}
.selectBox select {
  width: 100%;
  font-weight: bold;
}
.overSelect {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
.tags {
  display: none;
  border: 1px #dadada solid;
}
.tags label {
  display: block;
  display: flex;
  align-items: center;
  padding: 0 0.5rem;
}
.tags label:hover {
  background-color: #1e90ff;
}
.tags input {
  margin-right: 0.5rem;
}

/* .form--button.isValid {
  cursor: pointer;
  background-color: #fc8917;
}

.form--button.isValid:hover {
  opacity: 0.5;
} */

/* p > img {
  width: 100%;
} */
img[src^="data:image/*"] {
  width: 100%;
}

@media only screen and (min-width: 964px) {
	.blog {
  height: 275vh;
  height: 100%;
  padding: 1rem 1rem;
  max-width: 960px;
  margin: 0 auto;
  padding-top: 5rem;
  padding-left: 1rem;
  padding-right: 1rem;
}

.blog--items {
  margin-bottom: 15rem;
}

}
</style>