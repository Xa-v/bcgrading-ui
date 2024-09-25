<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';

// Function to unregister all service workers
async function clearServiceWorkerCache() {
  if ('serviceWorker' in navigator) {
    const registrations = await navigator.serviceWorker.getRegistrations();
    for (const registration of registrations) {
      registration.unregister();
    }
    console.log('Service workers unregistered and cache cleared.');
  } else {
    console.log('Service workers not supported in this browser.');
  }
}



// Function to clear all storage (cookies, localStorage, sessionStorage)
function clearStorage() {
  localStorage.clear();
  sessionStorage.clear();
  console.log('Local storage and session storage cleared.');
}

// Combined function to clear all cache and storage
function clearAllCache() {
  clearServiceWorkerCache();

  clearStorage();
}

// Optionally clear everything on page load
onMount(() => {
  // Uncomment if you want to clear everything automatically on page load
  clearAllCache();
});
</script>




<main>
    <div class="p-5 container-fluid">
      <header class="position-relative z-999">
        <div>
          <a href="/">
            <img src="
            https://benedictocollege.edu.ph/images/logo.png" alt="">
          </a>
        </div>
      </header>
      <div class="py-3 px-10">
        <div class="row">
          <div class="col-5 mt-5 left">
            <h1 class="text-left">Welcome to <span class="benedicto">Benedicto</span> <span class="college">College</span> Grading System</h1>
            <!-- <div class="text-center">
              <h1 class="w-100 benedicto-college">
                Welcome to Benedicto College
              </h1>
              <p>
                Management System
              </p>
            </div> -->
          <a href="/Login" type="button" class="btn btn-primary mt-4">Login to your account</a>
          </div>
          <div class="col-sm-7 px-0 d-none d-sm-block ">
            <img src="/src/lib/images/male-team-illustration.png" alt="" class="right-image">
          </div>
        </div>
      </div>
    </div>
</main>


<style>
  
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