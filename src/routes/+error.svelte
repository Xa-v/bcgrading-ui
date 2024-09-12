<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    function logout() {
    localStorage.removeItem('jwtToken');  // Clear the JWT token
    goto('/Login');  // Redirect to the login page immediately
    // window.location.href = '/Login';
  }



    onMount(() => {
        if (typeof window !== 'undefined' && window.history.length > 1) {
          
                window.history.back();
            
                // window.location.href = '/Login';
          
            
        } else {
            logout();
            // window.location.href = '/Login';
        }
    });
</script>

