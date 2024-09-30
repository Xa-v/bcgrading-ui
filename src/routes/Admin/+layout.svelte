<!-- src/routes/Admin/+layout.svelte -->
<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import { jwtDecode } from 'jwt-decode';
  import { fade } from 'svelte/transition';
 

  let error = '';
  let userRole = '';
  let userID = '';

  document.title = 'Class List - Grading System Portal';

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


<div class="d-flex">
  <!-- Sidebar -->
  <nav id="sidebar" class="topbar text-white d-flex flex-column custom-offcanvas-size p-3 vh-100" style="position: fixed; top: 0; left: 0; width: 250px; z-index: 1030;">
    <div class="d-flex align-items-center justify-content-between gap-2 mb-3">
      <div style="width: 48px; height: 48px;"></div>
    </div>

    <!-- Sidebar links -->
    <ul class="navbar-nav flex-column flex-grow-1">
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
        <li><a class="dropdown-item text-white p-0 py-1 " href="/Admin/Accounts">- View All users</a></li>
      
        <!-- <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="/" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            <img src="/src/lib/images/text-formatting-list-bullets.svg" alt="" class="box">
           Classes
           <img src="/src/lib/images/nav-arrow-down.svg" alt="">
          </a>
          <ul class="dropdown-menu bg-transparent">
            <li><a class="dropdown-item p-0 py-1" href="/Admin/SubjectList">- View All Subjects</a></li>
            <li class="nav-item">
              <a class="dropdown-item p-0 py-1" href="/Admin/ClassList" >
                
                List of Classes </a>
            </li>
      
          </ul>
        </li> -->
        <li><a class="dropdown-item text-white p-0 py-1 " href="/Admin/ClassList">-Class Lists </a></li>
        <li><a class="dropdown-item text-white p-0 py-1 " href="/Admin/SubjectList">-Subject Lists</a></li>
      
          </ul>
        
   

    <!-- Logout button -->
    <button type="button" class="btn btn-sm btn-danger p-2 mt-auto" data-bs-toggle="modal" data-bs-target="#staticBackdrop">
      Logout
    </button>
  </nav>

<!-- Main content area -->
<div class="app-container flex-grow-1" style="margin-left: 250px; padding-top: 20px;">
  <!-- Card or main body content -->
  <div class="card rounded-0" >
    <div class="card-body" style="height: 88.5vh;">     
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

  .custom-offcanvas-size {
    width: 15%; /* Adjust this percentage to control the size of the offcanvas */
    max-width: 15%; /* Ensures the offcanvas doesn't exceed this width */
    background-color: #001A56;

  }

  .topbar{
    background-color: #001A56;
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





