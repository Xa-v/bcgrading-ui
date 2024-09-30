<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';

// // Function to unregister all service workers
// async function clearServiceWorkerCache() {
//   if ('serviceWorker' in navigator) {
//     const registrations = await navigator.serviceWorker.getRegistrations();
//     for (const registration of registrations) {
//       registration.unregister();
//     }
//     console.log('Service workers unregistered and cache cleared.');
//   } else {
//     console.log('Service workers not supported in this browser.');
//   }
// }



// // Function to clear all storage (cookies, localStorage, sessionStorage)
// function clearStorage() {

//   sessionStorage.clear();
//   console.log('Local storage and session storage cleared.');
// }

// // Combined function to clear all cache and storage
// function clearAllCache() {
//   clearServiceWorkerCache();

//   clearStorage();
// }

// // Optionally clear everything on page load
// onMount(() => {
//   // Uncomment if you want to clear everything automatically on page load
//   clearAllCache();
// });
</script>




<main>
    <div class="p-5 container-fluid">
      <header class="position-relative z-999">
        <div class="logo_container">
          <a href="/">
            <img src="
            https://benedictocollege.edu.ph/images/logo.png" alt="Benedicto Logo" class="logo">
          </a>
        </div>
      </header>
      <div class="py-3 px-10">
        <div class="row">
          <div class="col-5 mt-5 left gwapo">
            <h1 class="text-left">Welcome to <span class="benedicto">Benedicto</span> <span class="college">College</span> Grading System</h1>
            <!-- <div class="text-center">
              <h1 class="w-100 benedicto-college">
                Welcome to Benedicto College
              </h1>
              <p>
                Management System
              </p>
            </div> -->
          <div class="button_container"><a href="/Login" type="button" class="btn btn_login btn-primary mt-4">Login to your account</a></div>
          </div>
          <div class="col-sm-7 px-0 d-none d-sm-block illustrator_container ">
            <img src="/src/lib/images/male-team-illustration.png" draggable="false" alt="" class="right-image">
          </div>
        </div>
      </div>
    </div>
</main>


<style>

  @media screen and (max-width: 557px)  {
    .text-left {
      font-size: 2rem;
      text-align: center;
      width: 100%;
    }

    .logo_container {
      width: 100%;
      position: relative
    }

    .logo_container a {
      width: auto;
    }
    
    .button_container { 
      position: relative;
      /* width: 100%; */
    }

    .btn_login {
      position: absolute;
      left: 20%;
      right: 20%;
      transform: translate(0%, 0);
    }


    .logo_container img {
      position: absolute;
      left: 70%;
      right: 50%;
      transform: translate(-60%, 0);
    }

    .illustrator_container {
      display: block;
    }



    .logo {
      margin-block: auto;
    }

    .gwapo {
      width: 100% !important;
    }
  }

  @media screen and (max-width: 376px)  {
    .logo_container img {
      position: absolute;
      left: 80%;
      right: 50%;
      transform: translate(-60%, 0);
    }
  }
  
  main{
    height: 100vh !important;
    margin: 0;
    overflow: hidden !important;
    width: 100%;
    
  }




   .right-image{
    width: 100%;
    height: 100%;
   }


   h1{
    font-size: 2.5rem;
    width: 100%;
    font-weight: 700;
   } 

   .benedicto{
    color: #0035AD;
    font-style: italic;
   }

   .college{
    color: #FE5C00;
    font-style: italic;
   }

</style>