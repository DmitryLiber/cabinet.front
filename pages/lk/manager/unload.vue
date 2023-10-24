<!-- eslint-disable vue/no-v-html -->
<template>
    <main class="unload">
        <h1 class="text-bold unload__title">Выгрузка</h1>

        <div v-if="!pending" class="unload__grid">
            <div class="unload__list">
                <Card>
                    <h4 class="text-bold card__title">Выгрузка</h4>
                    <b class="text-bold card__subtitle">Выполняет работу:</b>
                    <ul class="card__list">
                        <li class="card__item">Собирает фотографии из заказов пользователей.</li>
                        <li class="card__item">Выгружает по папкам</li>
                    </ul>
                </Card>
                <Card
                    v-for="{ id, task_date, status, status_text, event, size } in dataParser(archives)"
                    :key="id"
                    :class="`card_${status}`"
                    @click="() => getArchive(id)"
                >
                    <ul class="card__list card__list_alt">
                        <li class="card__item" v-html="task_date" />
                        <li class="card__item">
                            Статус задачи: <span class="text-bold">{{ status_text }}</span>
                        </li>
                        <li class="card__item">
                            ID выгрузки: <span class="text-bold">{{ id }}</span>
                        </li>
                        <li class="card__item">{{ event }}</li>
                        <li class="card__item">Размер выгрузки: {{ size }}</li>
                    </ul>
                </Card>
            </div>
            <div class="unload__info">
                <div v-if="archive === null" class="notice" data-color="light-purple">
                    Для того, чтобы посмотреть информацию о <b class="text-bold">выгрузке</b>, а также её скачать,
                    нажмите на требуемую выгрузку в столбце слева.
                </div>
                <div v-else class="block download-archives">
                    <b class="text-bold download-archives__title">Ссылка для скачивания архива Выгрузки (.zip):</b>

                    <div class="download-archives__buttons">
                        <a class="link" :href="archive['download_link']" download>{{ archive["download_link"] }}</a>
                        <button class="button button-text span-link" @click="() => copyUrl(archive['download_link'])">
                            скопировать ссылку
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </main>
</template>

<script setup lang="ts">
import { useAPIFetch } from "@composables/useAPIFetch";

const archive = ref(null);

const getArchive = id => {
    useAPIFetch(`e.scripts?page=pages:unload&event=get&unload_id=${id}`, {
        server: false,
        onResponse({ response }) {
            const newData = dataParser(response._data)[0];
            newData.download_link = `https://seenday.com/${linkParser(newData.download_link)}`;
            archive.value = newData;
        }
    });
};

const dataParser = (data: string) => JSON.parse(data).response.data;
const linkParser = (data: string) => data.split(":")[1].trim();
const { pending, data: archives } = useAPIFetch("e.scripts?page=pages:unload&event=get");
</script>

<script lang="ts">
export default {
    methods: {
        async copyUrl(url) {
            try {
                await navigator.clipboard.writeText(url);
                alert("Ссылка скопирована!");
            } catch ($e) {
                alert("Не удается скопировать ссылку...");
            }
        }
    }
};
</script>
