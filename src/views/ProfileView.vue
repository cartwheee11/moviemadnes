<script lang="ts" setup>
import { createGroup, editProfile, getProfile } from '@/api'
import { onMounted, ref, watch } from 'vue'
import ModalWindow from '@/components/ModalWindow.vue'
import type { Profile } from '../../types/shared'
import AvatarWithPlaceholder from '../components/AvatarWithPlaceholder.vue'
import AsyncButton from '@/components/AsyncButton.vue'
import ListItem from '@/components/ListItem.vue'
import { useRouter } from 'vue-router'

const router = useRouter()

//TODO: пагинация
//TODO: группы выводятся в обратном порядке
//TODO: УДАЛЕНИЕ ГРУППЫ

const groups = ref()
const isError = ref(false)
const createGroupModal = ref(false)
const groupName = ref('')
const profile = ref<Profile>()
const editProfileModal = ref<boolean>(false)
const editProfileInputs = ref({
  username: '',
  avatar: ''
})

watch(profile, () => {
  editProfileInputs.value = {
    username: profile.value?.username || '',
    avatar: profile.value?.avatar || ''
  }
})

function updateProfile(user: Profile) {
  profile.value = user
}

function onEditProfileButtonClick() {
  return new Promise<void>((resolve) => {
    editProfile({ name: editProfileInputs.value.username, avatar: editProfileInputs.value.avatar }).then(res => {
      if (res.data) {
        const { username, avatar, created_at, id } = res.data
        updateProfile({ username, avatar, id, created_at })

      }
      resolve()
      editProfileModal.value = false
    })
  })
}

function onCreateGroupButtonClick(): Promise<void> {
  return new Promise<void>((resolve => {
    createGroup(groupName.value).then((e) => {
      if (e.message == 'success') {
        window.history.go()

        createGroupModal.value = false
        groupName.value = ''
        resolve()
      } else {
        resolve()
      }
    })
  }))
}

onMounted(() => {
  getProfile().then(data => {
    if (data.message == 'success') {
      groups.value = data.data?.groups
      profile.value = data.data?.user
    } else {
      isError.value = true
    }

  })
})
</script>

<template>
  <ModalWindow @hide="createGroupModal = false" :visible="createGroupModal">
    <p><input v-model="groupName" class="input" type="text" placeholder="Название группы"></p>
    <p>
      <AsyncButton @click="() => onCreateGroupButtonClick()" class="btn w-full mt-4">создать</AsyncButton>
    </p>
  </ModalWindow>

  <ModalWindow @hide="editProfileModal = false" :visible="editProfileModal">
    <h3 class="font-black text-xl">Настройки профиля</h3>

    <fieldset class="fieldset w-full text-left mt-2">
      <legend class="fieldset-legend">Никнейм</legend>
      <input type="text" v-model="editProfileInputs.username" class="input w-full">
    </fieldset>

    <fieldset class="fieldset w-full text-left mt-2">
      <legend class="fieldset-legend">Ссылка на аватарку</legend>
      <input type="text" v-model="editProfileInputs.avatar" class="input w-full" placeholder="Описание">
    </fieldset>

    <AsyncButton class="mt-4 btn w-full" @click="() => onEditProfileButtonClick()">Отправить</AsyncButton>
  </ModalWindow>


  <section class="profile">
    <div class="container flex gap-4 lg:gap-10 items-center py-20 flex-col lg:flex-row">
      <AvatarWithPlaceholder :url="profile?.avatar" class="w-20 h-20 shrink-0 lg:w-40 lg:h-40 text-7xl">
        <span class="lg:text-7xl">{{ profile?.username[0].toUpperCase() }}</span>
      </AvatarWithPlaceholder>


      <div class="grow" v-if="!profile">
        <div class="skeleton w-full h-15"></div>
        <div class="skeleton w-50 h-7 mt-4"></div>
      </div>
      <div v-else class="about w-full flex flex-col items-center lg:items-start text-center lg:text-left">
        <h1 class="mb-3 font-black wrap-break-word max-w-50 md:max-w-full">{{ profile.username }}</h1>
        <p>
          <b>На сайте с </b>{{ new Date(profile.created_at as string).toLocaleString('ru-RU').split(',')[0] }}
        </p>
        <button @click="editProfileModal = true" class="btn mt-4 btn-info">
          <!-- Если пользователь меняет ник на уже существущий срабатывает internal error 500 -->
          <fa :icon="['fas', 'gear']" />
          Редактировать
        </button>
      </div>


    </div>

  </section>

  <section class="container !py-0 !pb-10">
    <h2>Мои группы</h2>
    <button @click="createGroupModal = true" class="add-table-row-button mt-4">+</button>

    <div v-if="!profile?.id" class="skeleton w-full h-100 mt-4"></div>
    <ListItem class="mt-4">
      <div v-for="(g) in groups" :key="g.id" class="cursor-pointer group flex items-center text-xl !p-6 gap-4"
        @click="router.push('/groups/' + g.id)">
        <strong class="flex gap-5 items-center lg:max-w-1/2">
          <AvatarWithPlaceholder class="w-10 h-10" :url="g?.avatar_url">{{ g.name[0] }}</AvatarWithPlaceholder>
          <span class="truncate max-w-45">{{ g.name }}</span>
        </strong>
        <div class="hide lg:show text-gray-400">—</div>
        <div class="absolute hide lg:show text-gray-400 truncate grow-1">{{ g.desc }}
        </div>
        <div class="absolute opacity-0 lg:relative lg:opacity-100 ">{{ new Date(g.created_at as
          string).toLocaleString('ru-RU').split(',')[0]
          }}
        </div>
      </div>
    </ListItem>
  </section>
</template>
