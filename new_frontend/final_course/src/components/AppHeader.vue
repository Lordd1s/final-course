<template>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark" style="z-index: 3;">
        <div class="container-fluid text-center">
            <span class="navbar-brand header-span">
                Добро пожаловать {{ auth.getUser.username }}!
            </span>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavDarkDropdown"
                aria-controls="navbarNavDarkDropdown" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNavDarkDropdown">
                <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                    <li class="nav-item" v-if="auth.getUser.isAdmin || auth.getUser.isMaster || auth.getUser.isStaff">
                        <RouterLink to="/profile" class="nav-link">Профиль</RouterLink>
                    </li>
                    <li class="nav-item">
                        <RouterLink to="/" class="nav-link">Главная</RouterLink>
                    </li>
                    <li class="nav-item">
                        <RouterLink to="/about" class="nav-link">О нас</RouterLink>
                    </li>
                    <li class="nav-item">
                        <RouterLink to="/vacancy" class="nav-link">Вакансии</RouterLink>
                    </li>
                    <li class="nav-item">
                        <RouterLink to="/newsletter" class="nav-link">Новости</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.isAdmin || auth.getUser.isStaff">
                        <RouterLink to="/all_cv" class="nav-link">Все резюме</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.isAdmin || auth.getUser.isStaff">
                        <RouterLink to="/create_vacancy" class="nav-link">Создать вакансию</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.isAdmin || auth.getUser.isStaff">
                        <RouterLink to="/applications" class="nav-link">Заявки</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.isMaster">
                        <RouterLink to="/all_cv" class="nav-link">Все резюме</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.isMaster">
                        <RouterLink to="/applications" class="nav-link">Заявка на трубы</RouterLink>
                    </li>
                </ul>

                <ul class="navbar-nav ml-auto">
                    <li class="nav-item" v-if="auth.getUser.length === 0">
                        <RouterLink to="/login" class="btn btn-outline-light">Войти</RouterLink>
                    </li>
                    <li class="nav-item" v-if="auth.getUser.length === 0">
                        <RouterLink to="/register" class="btn btn-outline-light">Зарегистрироваться</RouterLink>
                    </li>
                    <li class="nav-item" v-else>
                        <button @click="exit" class="btn btn-outline-danger">Выйти</button>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</template>
  
<script>
import { defineComponent, onMounted } from 'vue';
import { useAuthStore } from '@/store/AuthStore';
import axios from 'axios';
export default defineComponent({
    setup() {
        const AuthStores = useAuthStore()
        onMounted(() => {
            AuthStores.restoreUserInfo();

        });

        return { auth: AuthStores }
    },

    async mounted() {
        if (localStorage.getItem('refreshToken')) {
            setInterval(() => {
                this.refreshToken(localStorage.getItem('refreshToken'));
                console.log("Токены обновлены");
            }, 20 * 60 * 1000);
        }
    },

    methods: {
        exit() {
            localStorage.removeItem('accessToken');
            localStorage.removeItem('refreshToken');
            this.auth.logout();
        },
        async refreshToken(token) {
            try {
                const response = await axios.post(
                    `http://127.0.0.1:8000/api/token/refresh/`,
                    { "refresh": token },
                    {
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': `Bearer ${localStorage.getItem('accessToken')}`
                        }
                    }
                );
                ;
                // console.log(response);
                if (response.status == 200) {
                    localStorage.setItem('accessToken', response.data.access);
                    localStorage.setItem('refreshToken', response.data.refresh);
                }


            } catch (error) {
                console.error(error);
            }
        }
    },
});





</script>
  
<style scoped>
.header-span {
    font-size: 1.2rem;
}

.navbar-brand {
    font-weight: bold;
}

.nav-item {
    margin-right: 15px;
}

.nav-link {
    color: #fff !important;
}

.nav-link:hover {
    color: #c1e648 !important;
}

.btn-outline-light {
    color: #fff;
    border-color: #fff;
}

.btn-outline-light:hover {
    background-color: #fff;
    color: #000;
}

.btn-outline-danger {
    color: #dc3545;
    border-color: #dc3545;
}

.btn-outline-danger:hover {
    background-color: #dc3545;
    color: #fff;
}
</style>