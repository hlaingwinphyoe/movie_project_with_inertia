<template>
    <header class="border-b border-gray-800 shadow">
        <nav class="container mx-auto px-4 lg:px-6 py-1.5">
            <div class="flex justify-between items-center">
                <div class="flex items-center gap-3">
                    <ApplicationLogo class="h-8" />
                    <h4 class="text-xl uppercase font-bold">Movie</h4>

                    <div class="flex items-center">
                        <NavLink
                            href="/"
                            :active="route().current('dashboard')"
                        >
                            <span class="whitespace-nowrap text-sm">Movie</span>
                        </NavLink>
                        <NavLink
                            href="/"
                            :active="route().current('dashboard')"
                        >
                            <span class="whitespace-nowrap text-sm"
                                >About Us</span
                            >
                        </NavLink>
                        <NavLink
                            href="/"
                            :active="route().current('dashboard')"
                        >
                            <span class="whitespace-nowrap text-sm"
                                >Contact</span
                            >
                        </NavLink>
                    </div>
                </div>
                <div class="flex items-center gap-3">
                    <el-button circle>
                        <font-awesome-icon
                            :icon="['fas', 'magnifying-glass']"
                        />
                    </el-button>
                    <div v-if="canLogin">
                        <div v-if="auth.user">
                            <el-dropdown>
                                <span class="text-[28px]">
                                    <font-awesome-icon
                                        :icon="['fas', 'circle-user']"
                                    />
                                </span>
                                <template #dropdown>
                                    <el-dropdown-menu>
                                        <el-dropdown-item>
                                            <Link :href="route('dashboard')">
                                                Dashboard
                                            </Link>
                                        </el-dropdown-item>
                                        <el-dropdown-item
                                            >Profile</el-dropdown-item
                                        >
                                        <el-dropdown-item divided>
                                            <Link
                                                :href="route('logout')"
                                                as="button"
                                                method="post"
                                            >
                                                Sign Out
                                            </Link>
                                        </el-dropdown-item>
                                    </el-dropdown-menu>
                                </template>
                            </el-dropdown>
                        </div>

                        <div v-else>
                            <el-button
                                @click="handleLogin"
                                type="primary"
                                round
                                plain
                            >
                                Sign in
                            </el-button>
                        </div>
                    </div>
                </div>
            </div>
        </nav>
    </header>

    <Login :show="showLogin" @closed="closeDialog" />
</template>

<script>
import { onMounted, reactive, toRefs } from "vue";
import { initFlowbite } from "flowbite";
import ApplicationLogo from "@/Components/ApplicationLogo.vue";
import { Link, router, usePage } from "@inertiajs/vue3";
import NavLink from "@/Components/NavLink.vue";
import Login from "@/Pages/Auth/Login.vue";
export default {
    setup() {
        const state = reactive({
            canLogin: usePage().props.canLogin,
            auth: usePage().props.auth,
            showLogin: false,
        });

        onMounted(() => {
            initFlowbite();
        });

        const handleLogin = () => {
            state.showLogin = true;
        };

        const closeDialog = () => {
            state.showLogin = false;
        };

        return {
            ...toRefs(state),
            handleLogin,
            closeDialog,
        };
    },
    components: { ApplicationLogo, NavLink, Login, Link },
};
</script>

<style scoped>
.example-showcase .el-dropdown-link {
    cursor: pointer;
    color: var(--el-color-primary);
    display: flex;
    align-items: center;
}
</style>