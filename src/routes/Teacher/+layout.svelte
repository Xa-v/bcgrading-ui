<!-- src/routes/Admin/+layout.svelte -->
<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { jwtDecode } from 'jwt-decode';
    import { fade } from 'svelte/transition';
   
  

    let error = '';
    let showUnauthorizedMessage = false;
  
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
                // unauthorizedAccess("Log-in sa doy redirecting to login...");
                showUnauthorizedMessage = true;
                logout();
                      return;
                  }
  
                  try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;
            userID = decodedToken.id;
            console.log("Role:", userRole);
            console.log("ID:", userID);
            
  
            if (userRole !== 'Teacher') {
                // redirectMessage = `Role '${userRole}' does not have access to this page.`;
                // unauthorizedAccess("Redirecting you to your role-specific page.");
                showUnauthorizedMessage = true;
                goto(`/${userRole}`);
                return;
            }
  
          } catch (error) {
            console.error('Error:', error);
            // unauthorizedAccess("Error decoding token, redirecting to login.");
            showUnauthorizedMessage = true;
            logout();
        }
  
      
  
  
    });

    
  </script>
  {#if showUnauthorizedMessage}
  <div class="popup" >
   
      
  </div>
  {/if}
  
  
  {#if error}
      <p>{error}</p>
  {/if}
  
<!-- Top Navbar with white background, border, and shadow -->
<nav class="navbar navbar-light fixed-top p-0 shadow-none" style="border-bottom: 1px solid ;border-color: black;">
    <div class="navbar container-fluid">
      <div class="d-flex align-items-center gap-2">
        <!-- Navbar branding or logo if needed -->
      </div>
      <div>
        <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
      </div>
    </div>
  </nav>
  
  
  <div class="d-flex">
    <!-- Sidebar -->
    <nav id="sidebar" class="topbar text-white d-flex flex-column custom-offcanvas-size p-3 vh-100" style="position: fixed; top: 0; left: 0; width: 250px; z-index: 1030;">
      <div class="d-flex align-items-center justify-content-between gap-2 mb-3">
        <div style="width: 48px; height: 48px;"></div>
      </div>
  
      <!-- Sidebar links -->
      <ul class="navbar-nav flex-column flex-grow-1">
        <li class="nav-item d-flex align-items-center gap-4">
          <a class="nav-link active text-white" href="/Teacher">
            <img src="/src/lib/images/home.svg" alt="">
            Home
          </a>
        </li>
        <li class="nav-item d-flex align-items-center gap-4">
            <a class="nav-link active text-white" href="/Teacher/MyClass">
              <img src="/src/lib/images/home.svg" alt="">
              My Classes
            </a>
          </li>
      </ul>
  
      <!-- Logout button -->
      <button type="button" class="btn btn-sm btn-danger p-2 mt-auto" data-bs-toggle="modal" data-bs-target="#staticBackdrop">
        Logout
      </button>
    </nav>
  
  <!-- Main content area -->
<div class="app-container flex-grow-1" style="margin-left: 250px; padding-top: 80px;">
    <!-- Card or main body content -->
    <div class="card rounded-0" >
      <div class="card-body" style="height: 88vh;">
        <slot></slot>
      </div>
    </div>
  </div>
  
  </div>
  
  
  
  <!-- Button trigger modal -->
  
  
  <!-- Modal logout-->
<div class="modal fade" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered modal-md">
      <div class="modal-content ">
        
        <div class="modal-body">
          <p class="text-center fw-bold fs-5">You are logging out of Benedicto College Grading System.</p>
          <p class="text-center mt-2 text-body-secondary">Are you sure you want to log out?</p>
      </div>
        <div class="modal-footer">
        
    <div class="w-100">
          <div class="d-flex justify-content-center align-items-center gap-2">
            <button type="button" class="btn  btn-danger  w-100" on:click={logout}>Yes</button>
            <button type="button" class="btn  btn-secondary w-100" data-bs-dismiss="modal">No </button>
          </div>
      </div>
    </div>
      </div>
    </div>
  </div>
  
  
  
  <style>
   .popup {
          position: fixed;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background-image: url('/src/lib/images/crying-cat-thumb.jpg');
          /* background-color: rgb(0, 0, 0,0); */
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
          color: white;
          display: flex;
          justify-content: center;
          align-items: center;
          z-index: 1050;
      }
  
  
    .custom-offcanvas-size {
      width: 15%; /* Adjust this percentage to control the size of the offcanvas */
      max-width: 15%; /* Ensures the offcanvas doesn't exceed this width */
      background-color: #001A56;
  
    }
  
    /* .custom-navbar-size {
      height: 10%; 
      max-height: 10%;
    
    } */
  
    /* .dropdown-menu{
      border: none !important;
    } */
  
    /* .navbar{
      background-color: #001A56;
    }  */
    .topbar{
      background-color: #001A56;
    }
  
    /* .dropdown-item:hover{
      background-color: transparent !important;
      color: #FF6100 !important;
    } */
  
    /* .logo{
    
      width: 100%;
      height: 30px;
    
    } */
  
    .navbar-button{
      outline: none;
      border: none;
    }
  
    .profile{
      width: 40px;
      height: 40px;
      filter: invert(1);
    }
  
    li > a > img{
      filter: invert(1);
    }
  
    li > a{
      color: white;
      display: flex;
      gap: 1rem;
      
    }
  
    a::after{
      display: none;
  
    }
  
    a:hover{
      color: #FF6100 !important;
    }
  
    .box{
      height: 18px;
      margin-left: 3px;
      width: 18px;
    }
  </style>
  