<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
    let error = '';
    let userRole = '';
    
    $: ({ classid } = $page.params);

    onMount(async () => {
    
      sessionStorage.clear();
       await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

        
      
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;  // Save userRole for later use

            // Fetch the years if the user is authorized
            const classdetails = await fetch(`http://localhost:4000/registrar/classinfo/${classid}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (classdetails.ok) {
                classinfo = await classdetails.json();
            } else {
                error = `Failed to fetch classdetails: ${classdetails.statusText}`;
            }


       
    });

</script>

{#if classinfo}
<div class="mb-3">
    <a class="btn btn-sm btn-outline-secondary " href={`/Registrar/ClassList`}>
      ← 
    </a>
  </div>
    <!-- Card with Class Info -->
    <div class="card rounded-0 shadow-sm mb-2">
      <div class="card-body">
        <!-- Header of Class Info -->
        <h2 class="card-title mb-3">
          {classinfo.Subjectitle?.title} ({classinfo.subjectcode})
        </h2>
        <div class="d-flex justify-content-start gap-4 mb-1">
            <p class="text-muted mb-0"><strong>Year: </strong> <strong> {classinfo.year}</strong></p>
            <p class="text-muted mb-0"><strong>Semester: </strong><strong>{classinfo.semester}</strong></p>
            <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName} </strong> </p>
          </div>
        

      <ul class="nav nav-tabs rounded-0 " id="myTab" role="tablist">
          <li class="nav-item">
            <a class="nav-link rounded-0 " href={`/Registrar/${classid}/StudentList`} role="tab" aria-selected="false">Students List</a>
          </li>
          <li class="nav-item">
            <a class="nav-link rounded-0" href={`/Registrar/${classid}/AddStudent`} role="tab" aria-selected="false">Add Student</a>
          </li>
          <li class="nav-item">
            <a class="nav-link active disabled rounded-0" href={`/Registrar/${classid}`} role="tab" aria-selected="true">Scores</a>
          </li>
        </ul>



      </div>

    </div>



    <!-- Second Card with Nav Tabs -->
    <!-- <div class="card rounded-0 shadow-sm mb-4">
      <div class="card-body">
    
    </div>
  </div> -->
{/if}

{#if error}
  <div class="alert alert-danger mt-4">
    {error}
  </div>
{/if}


<style>
  .nav-link:hover {
      background-color: #001A56 !important; /* Bootstrap primary color or any custom color */
      color: rgb(255, 255, 255) !important; /* Make the text white when hovered */
      transition: background-color 0.2s ease-in-out !important; /* Smooth transition */
      
  }

  .active {
        background-color: #001A56 !important; /* Make the background of navtab blue when active */
        color: rgb(255, 255, 255) !important; /* Make the text white when active */
    }
</style>