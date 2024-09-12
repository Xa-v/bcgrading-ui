<!-- src/routes/Admin/+layout.svelte -->
<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import { jwtDecode } from 'jwt-decode';
  import { fade } from 'svelte/transition';
 

  let offcanvasElement;
  let bootstrap;  // Variable to hold the imported Bootstrap module
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
          

          if (userRole !== 'Admin') {
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

    
    if (typeof window !== 'undefined') {
      // Dynamically import Bootstrap's JS only in the browser
      bootstrap = await import('bootstrap/dist/js/bootstrap.bundle.min.js');

      // Handle click events on nav links to close the offcanvas
      const navLinks = document.querySelectorAll('.navbar-nav .dropdown-item .nav-link:not(.dropdown-toggle)');
      navLinks.forEach(link => {
        link.addEventListener('click', () => {
          if (offcanvasElement) {
            const offcanvasInstance = bootstrap.Offcanvas.getInstance(offcanvasElement);
            if (offcanvasInstance) {
              offcanvasInstance.hide();
            }
          }
        });
      });

      // Add event listener to prevent closing the offcanvas when clicking inside the dropdown
      const dropdownMenus = document.querySelectorAll('.dropdown-menu-no-close');
      dropdownMenus.forEach(menu => {
        menu.addEventListener('click', (event) => {
          event.stopPropagation();
        });
      });
    }


  });
 // Function to handle closing offcanvas
 function closeOffcanvas() {
    if (offcanvasElement && bootstrap) {
      const offcanvasInstance = bootstrap.Offcanvas.getInstance(offcanvasElement);
      if (offcanvasInstance) {
        offcanvasInstance.hide();
      }
    }
  }

  // Reactively close offcanvas on route change
  $: $page.url, closeOffcanvas();



 

  
</script>
{#if showUnauthorizedMessage}
<div class="popup" >
 
    
</div>
{/if}


{#if error}
    <p>{error}</p>
{/if}

<nav class="navbar navbar fixed-top custom-navbar-size p-0">
  <div class="container-fluid">

    <div class="d-flex align-items-center gap-2">

      <button class="navbar-toggler bg-transparent navbar-button" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasDarkNavbar">
        <img src="https://assets.website-files.com/62e8f5c9dbfdccaf94d287ac/62ea9a21af559238cc12a830_menu_FILL0_wght400_GRAD0_opsz48%20(2).svg" alt="" class="navbar-toggler-icon navbar-span">
      </button>

      
    </div>
    <div>
      <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
    </div>
    
    <!-- Offcanvas Menu -->
    <div bind:this={offcanvasElement} class="offcanvas offcanvas-start text-white custom-offcanvas-size" tabindex="-1" id="offcanvasDarkNavbar">
      <div class="offcanvas-header">
        <div class="d-flex align-items-center justify-content-between gap-2">
          <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
       
        </div>
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="offcanvas" aria-label="Close">
         
        </button>
      </div>
      <div class="offcanvas-body d-flex flex-column justify-content-between">
        <nav>
          <ul class="navbar-nav justify-content-end flex-grow-1 pe-3 ">
            <li class="nav-item d-flex alig-items-center gap-4">
              <a class="nav-link active text-white" href="/Admin">
                <img src="/src/lib/images/home.svg" alt="">
                Home
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/Admin/AddAccount/">
                <img src="/src/lib/images/th-large.svg" alt="" class="box">Add user</a>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="/" role="button" data-bs-toggle="dropdown" aria-expanded="false">
              
                Manage Users

                  <img src="/src/lib/images/nav-arrow-down.svg" alt="">
              </a>
              <ul class="dropdown-menu bg-transparent">
                <li><a bind:this={offcanvasElement} class="dropdown-item text-white p-0 py-1 " href="/Admin/Accounts">- View All users</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item text-white p-0 py-1" href="/Admin/UpdateAccount">- Update User Info</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item text-white p-0 py-1" href="/Admin/UpdatePassword">- Change Passwords</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item text-white p-0 py-1" href="/Admin/DeleteAccount">- Deactivate User</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item text-white p-0 py-1" href="/Admin/ReactivateAccount">- Restore User</a></li>
          
              </ul>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="/" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                <!-- <img src="/src/lib/images/text-formatting-list-bullets.svg" alt="" class="box"> -->
               Classes
               <img src="/src/lib/images/nav-arrow-down.svg" alt="">
              </a>
              <ul class="dropdown-menu bg-transparent">
                <li><a bind:this={offcanvasElement} class="dropdown-item p-0 py-1" href="/Admin/SubjectList">- View All Subjects</a></li>
                <li class="nav-item">
                  <a bind:this={offcanvasElement} class="dropdown-item p-0 py-1" href="/Admin/ClassList" >
                    
                    List of Classes </a>
                </li>
                <!-- <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/updateAccount">Update User Info</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/updatePassword">Change Passwords</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/deleteAccount">Deactivate User</a></li>
                <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/reactivateAccount">Restore User</a></li> -->
          
              </ul>
            </li>
            <li>
              <hr class="dropdown-divider">
            </li>
          </ul>
        </nav>
      
        
        <button type="button" class="btn btn-sm btn-danger p-2" data-bs-toggle="modal" data-bs-target="#staticBackdrop">
          Logout 
           </button>
      </div>
    </div>
  </div>
</nav>

<div class="app-container">
  <nav class="navbar navbar-expand navbar-dark bg-dark hover">
    <div class="container">
      <div class="navbar-nav me-auto">
   
      </div>
      <div class="navbar-nav ms-auto">
        <button class="nav-item nav-link text-white">LHAHAHAHA</button>
      </div>
    </div>
  </nav>
  <div class="card">
    <div class="card-header"></div>
    <div class="card-body">
      <slot></slot>
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

  .custom-navbar-size {
    height: 10%; /* Adjust this percentage to control the size of the offcanvas */
    max-height: 10%;
  
  }

  .dropdown-menu{
    border: none !important;
  }

  .navbar{
    background-color: #001A56;
  }

  .dropdown-item:hover{
    background-color: transparent !important;
    color: #FF6100 !important;
  }

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
