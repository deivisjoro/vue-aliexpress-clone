<script setup>

    const client = useSupabaseClient();
    const user = useSupabaseUser();

    const login = async (provider) => {

        const { data, error } = await client.auth.signInWithOAuth({
            provider,
            
        })

    }

    watchEffect(()=>{
        if(user.value){
            return navigateTo('/');
        }
    })

</script>

<template>
    <div id="AuthPage" class="w-full h-[100vh] bg-white">
        <div class="w-full flex items-center justify-center p-5 border-b border-b-gray-300">
            <NuxtLink to="/">
                <img src="/aliexpress-logo.png" alt="logo aliexpress" width="170">
            </NuxtLink>
        </div>

        <div class="max-w-[400px] mx-auto px-2">
            <div class="text-center my-6">
                Iniciar Sesion / Registrarse
            </div>
            <button class="flex items-center justify-center gap-3 p-1.5 w-full border hover:bg-gray-100 rounded-full text-lg font-semibold" @click="login('google')">
                <img src="/google-logo.png" alt="logo google" class="w-full max-w-[30px]" />
                <div>Google</div>
            </button>
            <button class="flex items-center justify-center gap-3 p-1.5 w-full border hover:bg-gray-100 rounded-full text-lg font-semibold mt-4" @click="login('github')">
                <img src="/github-logo.png" alt="logo google" class="w-full max-w-[30px]" />
                <div>Github</div>
            </button>

        </div>
    </div>
</template>