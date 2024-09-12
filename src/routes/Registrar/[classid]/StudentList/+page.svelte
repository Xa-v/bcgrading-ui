<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
    let students = [];
    let error = '';
    let userRole = '';
    
    $: ({ classid } = $page.params);

    onMount(async () => {

       // Import Bootstrap JS
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

            const studentlist = await fetch(`http://localhost:4000/registrar/studentlist/${classid}`, {
            headers: {
                'Authorization': `Bearer ${token}` // Include JWT token
            }
        });

        if (studentlist.ok) {
            const studentData = await studentlist.json();
            students = studentData.students; // Update students with the correct data structure
        } else {
            error = `Failed to fetch studentlist: ${studentlist.statusText}`;
        }


       
    });

</script>

{#if classinfo}
<div class="mb-3">
    <button class="btn btn-sm btn-outline-secondary " on:click={() => window.history.back()}>
      ← 
    </button>
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
        

          

      </div>
    </div>



    <!-- Second Card with Nav Tabs as Links -->
<div class="card rounded-0 shadow-sm mb-4">
    <div class="card-body">
      <!-- Nav Tabs as Links -->
      <ul class="nav nav-tabs rounded-0 " id="myTab" role="tablist">
        <li class="nav-item">
          <a class="nav-link  active rounded-0 border" href={`/Registrar/${classinfo.classid}/StudentList`} role="tab" aria-selected="true">Students List</a>
        </li>
        <li class="nav-item">
          <a class="nav-link rounded-0" href={`/Registrar/${classinfo.classid}/AddStudent`} role="tab" aria-selected="false">Add Student</a>
        </li>
        <li class="nav-item">
          <a class="nav-link rounded-0" href={`/Registrar/${classinfo.classid}`} role="tab" aria-selected="false">Scores</a>
        </li>
      </ul>
    
   <!-- Display Students List Inside a Bordered Bootstrap Table -->
   <table class="table w-auto m-0 table-bordered border-dark">
    <thead>
        <tr>
            <th scope="col" class="p-2 text-white">Student ID</th>
            <th scope="col" class="p-2 text-white">Student Name</th>
            <th scope="col" class="p-2 text-white">Student Name</th>
           
          </tr>
      <tr>
        <th scope="col" class="p-2">Student ID</th>
        <th scope="col" class="p-2">Last Name</th>
        <th scope="col" class="p-2">First Name</th>
       
      </tr>
    </thead>
    <tbody>
      {#each students as student}
        <tr>
            <td>{student.studentinfo?.id}</td>
          <td>{student.studentinfo?.lastName} </td>
          <td>{student.studentinfo?.firstName}</td>
         
        </tr>
      {/each}
    </tbody>
  </table>
  
  
   
 </div>
</div>
  
{/if}

{#if error}
  <div class="alert alert-danger mt-4">
    {error}
  </div>
{/if}