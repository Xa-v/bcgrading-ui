<!-- <script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import { jwtDecode } from 'jwt-decode';

    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';
    let userRole = '';
    let showLogoutConfirm = false;

    onMount(() => {
        const token = localStorage.getItem('jwtToken');

        if (!token) {
            // No token found, redirect to login page
            unauthorizedAccess("No token found, redirecting to login.");
            return;
        }

        try {
            const decodedToken = jwtDecode(token);
         

            userRole = decodedToken.role;  // Save userRole for later use
         

            // Check if the user has the correct role to access the page
            if (userRole !== 'Teacher') {
                // Unauthorized access, redirect to the page for the current user role
                redirectMessage = `Role '${userRole}' does not have access to this page.`;
                unauthorizedAccess("Redirecting you to your role-specific page.");
            }
        } catch (error) {
           
            unauthorizedAccess("Error decoding token, redirecting to login.");
        }
    });

    function unauthorizedAccess(message) {
        console.error(message); // Debugging line
        showUnauthorizedMessage = true;
        redirectMessage = message;
        const interval = setInterval(() => {
            countdown--;
            if (countdown <= 0) {
                clearInterval(interval);
                if (redirectMessage.includes("login")) {
                    goto('/login');  // Redirect to login page
                } else {
                    goto(`/${userRole}`);  // Redirect to User Role
                }
            }
        }, 1000);
    }

    function logout() {
    localStorage.removeItem('jwtToken');  // Clear the JWT token
    goto('/login');  // Redirect to the login page immediately
}

function showConfirmLogout() {
        showLogoutConfirm = true;
    }

    function confirmLogout() {
        showLogoutConfirm = false;
        logout();
    }

    function cancelLogout() {
        showLogoutConfirm = false;
    }
</script>

<h1>Teacher Dashboard</h1>
<p>Welcome, Teacher!</p>

<button on:click={showConfirmLogout} >Logout</button>

{#if showLogoutConfirm}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>Are you sure you want to log out?</p>
        <button on:click={confirmLogout}>Yes</button>
        <button on:click={cancelLogout}>No</button>
    </div>
</div>
{/if}


{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>{redirectMessage}</p>
        <p>Redirecting you in {countdown} seconds...</p>
    </div>
</div>
{/if}



<style>
    .popup {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.9);
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 1000;
    }

    .popup-content {
        padding: 20px;
        background-color: rgba(255, 255, 255, 0.1);
        border-radius: 8px;
        text-align: center;
    }

    .popup-content button {
        margin: 5px;
        padding: 10px 20px;
        font-size: 16px;
        border: none;
        border-radius: 1px;
        cursor: pointer;
    }

    .popup-content button:first-of-type {
        background-color: #4CAF50; /* Green for Yes */
        color: white;
    }

    .popup-content button:last-of-type {
        background-color: #f44336; /* Red for No */
        color: white;
    }
</style> 
-->

<script>
      onMount(async () => {
        sessionStorage.clear();

      });
</script>
<h1>Teacher Dashboard</h1>
<p>Welcome, Teacher!</p> 