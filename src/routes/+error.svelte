<!-- <script context="module">
    export async function load({ error, status }) {
        return {
            props: {
                status,
                errorMessage: error.message || 'Not Found'
            }
        };
    }
</script> -->

<script>
    import { onMount } from 'svelte';

    onMount(() => {
        if (typeof window !== 'undefined' && window.history.length > 1) {
            setTimeout(() => {
                window.history.back();
                }, 75); 
            
        } else {
         
            window.location.href = '/';
        }
    });
</script>

<div class="error-page">
 
</div>

<style>
    .error-page {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-image: url('https://img.freepik.com/premium-photo/woman-with-scary-eyes-scary-face-with-eyes_869640-52219.jpg');
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
      
    }

    
</style>
