<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
    let attendanceprelim = [];
    let attendancescoresprelim = [];
    let error = '';
    let userRole = '';
    
    $: ({ classid, gradeid } = $page.params);

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

            // const prelimsattendance = await fetch(`http://localhost:4000/teacher/Prelim/Attendance/${classid}`, {
            //     headers: {
            //         'Authorization': `Bearer ${token}` // Include JWT token
            //     }
            // });

            // if (prelimsattendance.ok) {
            //     attendanceprelim = await prelimsattendance.json();
            // } else {
            //     error = `Failed to fetch prelimsattendance: ${prelimsattendance.statusText}`;
            // }
            const attendancescores = await fetch(`http://localhost:4000/teacher/getgradelist/${gradeid}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (attendancescores.ok) {
                attendancescoresprelim = await attendancescores.json();
            } else {
                error = `Failed to fetch attendancescores: ${attendancescores.statusText}`;
            }

       
    });


</script>

{#if classinfo}
<div class="mb-3">
    <button class="btn btn-sm btn-outline-secondary "  on:click={() => window.history.back()}>
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
        
           <!-- Nav Tabs as Links -->
           <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
            <li class="nav-item flex-grow-1">
                <a class="nav-link active disabled rounded-0  text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="true">PRELIM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0  text-center" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0  text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
            </li>
        </ul>

        <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
          <li class="nav-item flex-grow-1">
              <a class="nav-link active disabled rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Attendance`} role="tab" aria-selected="true">ATTENDANCE</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Participation`} role="tab" aria-selected="false">Participation</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Prelim/Quiz`} role="tab" aria-selected="false">QUIZ</a>
          </li>
          <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Prelim/Activity-Project`} role="tab" aria-selected="false">ACTIVITY/PROJECT</a>
        </li>
        <li class="nav-item flex-grow-1">
          <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Prelim/Exam`} role="tab" aria-selected="false">EXAM</a>
      </li>
      </ul>


      
      <button class="rounded-0 btn btn-outline-primary d-flex justify-content-center align-items-center mt-2 mb-2" style="height: 25px;" on:click={() => window.history.back()}>
        ←
    </button>
      
         <!-- Cards displaying attendance data -->
         <div style="max-height: 50vh; overflow-y: auto; overflow-x: hidden;">
            <table class="table table-bordered table-hover">
            
                <thead class="table-light">
                    
                    <tr>
                   
                        <!-- <th scope="col">Grade ID</th> -->
                        <th scope="col">Term</th>
                        <th scope="col">Score Type</th>
                        <th scope="col">Score</th>
                        <th scope="col">Perfect Score</th>
                        <th scope="col">Last Name</th>
                        <th scope="col">First Name</th>
                        <th scope="col">Student ID</th>
                    </tr>
                </thead>
                <tbody>
                    {#each attendancescoresprelim as attendance}
                    <tr>
                        <!-- <td>{attendance.gradeid}</td> -->
                        <td>{attendance.term}</td>
                        <td>{attendance.scoretype}</td>
                        <td>{attendance.score}</td>
                        <td>{attendance.perfectscore}</td>
                        <td>{attendance.Studentlist?.studentinfo?.lastName}</td>
                        <td>{attendance.Studentlist?.studentinfo?.firstName}</td>
                        <td>{attendance.Studentlist?.studentinfo?.id}</td>
                    </tr>
                    {/each}
                </tbody>
            </table>
        </div>
        
        


    </div>
      </div>

    {/if}


    <!-- Second Card with Nav Tabs as Links -->
    <!-- <div class="card rounded-0 shadow-sm mb-4">
        <div class="card-body">
        </div> 
    </div> -->
    
      


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