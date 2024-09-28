<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';


   
    let subjects = [];
   
    let error = '';
  
 
   

    onMount(async () => {

       // Import Bootstrap JS
       await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

         

            const subjectlist = await fetch('http://localhost:4000/registrar/subjects', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (subjectlist.ok) {
                subjects = await subjectlist.json();
            } else {
                error = `Failed to fetch subjects: ${subjectlist.statusText}`;
            }
                 
    });




    function clearFilters() {
    selectedYear = '';
    selectedSemester = '';
    selectedSubject = '';
    filterClasses();
}



</script>

<!-- <div class="container text-center">
    <div class="btn-group me-2">
       
    </div>

    <div class="btn-group me-2">
    
    </div>

    <div class="btn-group">
     
    </div>
    <button type="button" class="btn text-white btn-dark" on:click={clearFilters}>Clear Filters <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16">
        <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/>
      </svg></button>
</div> -->






<h2>Subject List</h2>

{#if error}
    <p>{error}</p>
{/if}


  
            <!-- Show filtered table if there are results -->
            <div style="max-height: 73vh; overflow-y: auto;">
                <table class="table table-bordered" style="width: 100% !important;">
                <thead>
                    <tr>
                       
                        <th>Subject Code</th>
                        <th>Subject Title</th>

                    </tr>
                </thead>
                <tbody>
                    {#each subjects as subject}
                    <tr>
                        <td>{subject.subjectcode}</td>
                        <td>{subject.title}</td>                
                    </tr>
                    {/each}
                </tbody>
            </table>
            </div>
      





<style>
  
   
</style>
