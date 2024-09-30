<!-- src/routes/Admin/+layout.svelte -->
<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { jwtDecode } from 'jwt-decode';
    import { fade } from 'svelte/transition';
   
    document.title = 'Admin Dashboard - Grading System Portal';
  
    let error = '';
    let userRole = '';
    let userID = '';
  
    // Function to handle logout
    function logout() {
      localStorage.removeItem('jwtToken');  // Clear the JWT token
      goto('/Login');  // Redirect to the login page immediately
    }
  
  
  
  
  
    // Use onMount to handle client-side operations
    onMount(async () => {
  
      await import('bootstrap/dist/js/bootstrap.bundle.min.js');
  
  
     
        const token = localStorage.getItem('jwtToken');
        // const idtoken = jwtDecode(token);
        //  userID = idtoken.id;
        // console.log("User ID:", userID);
  
          if (!token) {               
                logout();
                      return;
                  }
  
                  try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;
            userID = decodedToken.id;
            console.log("Role:", userRole);
            console.log("ID:", userID);
            
  
            if (userRole !== 'Admin') {

                goto(`/${userRole}`);
                return;
            }
  
          } catch (error) {
            console.error('Error:', error);
            logout();
        }
  
      
    });
  
    
  </script>

  
  {#if error}
      <p>{error}</p>
  {/if}

  
 
  <h1 class="text-center">Admin ACCOUNT</h1>
  <p class="text-center">Welcome, Admin!</p>
 
  
  


