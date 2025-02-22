<template>
    <div ref="navbarDiv" class="text-sm">
        <div id="navbarMenu" ref="navbarMenu"
             class="invisible opacity-0 fixed top-32 -right-2.5 bg-white py-2.5 px-5 rounded-2xl w-52 duration-500">
            <ul>
                <NavbarMenuLink v-if="userId" id="profileMenuOption" :alt="content.profile" :page-lang="langPrefix"
                                image-src="images/icons/user.webp"
                                page-name="lang-profile" @click="menuToggle(false)">
                    {{ content.profile }}
                </NavbarMenuLink>

                <NavbarMenuLink v-if="userId" id="settingsMenuOption" :alt="content.settings" :page-lang="langPrefix"
                                image-src="images/icons/settings.webp"
                                page-name="lang-settings" @click="menuToggle(false)">
                    {{ content.settings }}
                </NavbarMenuLink>

                <NavbarMenuLink v-if="userId && role === 'company'" id="companyMenuOption" :alt="content.company"
                                :page-lang="langPrefix"
                                image-src="images/icons/company.webp" page-name="lang-company"
                                @click="menuToggle(false)">
                    {{ content.company }}
                </NavbarMenuLink>

                <NavbarMenuLink id="catalogMenuOption" :alt="content.catalog"
                                :page-lang="langPrefix"
                                image-src="images/icons/catalog.webp" page-name="lang-catalog"
                                @click="menuToggle(false)">
                    {{ content.catalog }}
                </NavbarMenuLink>

                <NavbarMenuLink v-if="userId" id="cartMenuOption" :alt="content.cart" :page-lang="langPrefix"
                                image-src="images/icons/cart.png"
                                page-name="lang-cart" @click="menuToggle(false)">
                    {{ content.cart }}
                </NavbarMenuLink>

                <NavbarMenuLink v-if="userId" id="ticketsMenuOption" :alt="content.tickets" :page-lang="langPrefix"
                                image-src="images/icons/support.webp"
                                page-name="lang-tickets" @click="menuToggle(false)">
                    {{ content.tickets }}
                </NavbarMenuLink>

                <NavbarMenuLink v-if="userId" id="feedbackMenuOption" :alt="content.feedback" :page-lang="langPrefix"
                                image-src="images/icons/feedback.webp"
                                page-name="lang-feedback" @click="menuToggle(false)">
                    {{ content.feedback }}
                </NavbarMenuLink>

                <NavbarMenuFunction v-if="userId" id="logoutMenuOption" :alt="content.disconnect" :fun="logoutUser"
                                    image-src="images/icons/logout.webp" @click="menuToggle(false)">
                    {{ content.disconnect }}
                </NavbarMenuFunction>

                <NavbarMenuLink v-if="!userId" id="loginMenuOption" :alt="content.login" :page-lang="langPrefix"
                                :page-redirect="route.path"
                                image-src="images/icons/logout.webp" page-name="lang-login" @click="menuToggle(false)">
                    {{ content.login }}
                </NavbarMenuLink>
            </ul>
        </div>
        <div id="user" class="cursor-pointer" @click="menuToggle()">
            <NuxtImg id="profileImage" alt="Profile picture" class="navbar-icon rounded-[50%]" height="50px"
                     v-bind:src="imageSrc" width="50px" />
        </div>
    </div>
</template>

<script lang="ts" setup>
import { isLogged, loggedIn, logout } from "public/ts/checkLogin";

const nuxtApp = useNuxtApp();
const route = useRoute();

const langPrefix = nuxtApp.$rawLangPrefix;
const content = nuxtApp.$content.navBar;
const userId = ref<number | undefined | null>(null);
const role = ref<string | null>("");
const imageSrc = ref(nuxtApp.$profilePicture.url);

const divRef = useTemplateRef("navbarDiv");
const menuRef = useTemplateRef("navbarMenu");

onMounted(async () => {
    document.addEventListener("click", handleClickOutside);

    userId.value = await isLogged();

    role.value = localStorage.getItem("role");

    await getUSerPicture();
});

onBeforeUnmount(() => {
    document.removeEventListener("click", handleClickOutside);
});

function handleClickOutside(event: MouseEvent) {
    if (divRef.value) {
        const isClickedOnMenu = divRef.value.contains(event.target as Node);
        if (!isClickedOnMenu) menuToggle(false);
    }
}

async function logoutUser() {
    await fetch("https://api.ardeco.app/logout", {
        method: "GET",
        credentials: "include"
    });
    imageSrc.value = nuxtApp.$changeProfilePicture().url;
    logout();
    location.reload();
}

async function getUSerPicture() {
    if (!loggedIn) {
        // If not logged in, reset the profile picture cookie
        imageSrc.value = nuxtApp.$changeProfilePicture().url;
        return;
    }

    const response = await fetch(`https://api.ardeco.app/profile_picture/user`, {
        method: "GET",
        headers: {
            "Content-Type": "application/json"
        },
        credentials: "include"
    });
    const result = await response.json();
    if (imageSrc.value !== result.data.url) imageSrc.value = result.data.url;
    nuxtApp.$changeProfilePicture(result.data);
}

function menuToggle(activate: boolean | null = null) {
    if (menuRef.value) {
        if (activate !== null) {
            if (activate) menuRef.value.classList.remove("invisible");
            else menuRef.value.classList.add("invisible");
        } else {
            menuRef.value.classList.toggle("invisible");
        }
    }
}
</script>

<style lang="scss" scoped>
#navbarMenu {
    box-shadow: -5px 5px 10px 0px rgba(0, 0, 0, .1);
}

#navbarMenu:not(.invisible) {
    top: 80px;
    opacity: 1;
}
</style>
