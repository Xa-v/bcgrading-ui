<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { onMount } from 'svelte';
  import {jwtDecode} from 'jwt-decode';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  let classinfo = [];
  let attendancemidterm = [];
  let error = '';
  let successMessage = '';
  let userRole = '';

  let term = 'Midterm';  // Default to Prelim
  let scoretype = 'Attendance';  // Default to Attendance
  let attendanceDate = '';

  
  $: ({ classid, gradeid } = $page.params);

  onMount(async () => {
    sessionStorage.clear();
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

          const midtermsattendance = await fetch(`http://localhost:4000/teacher/Midterm/Attendance/${classid}`, {
              headers: {
                  'Authorization': `Bearer ${token}` // Include JWT token
              }
          });

          if (midtermsattendance.ok) {
            attendancemidterm = await midtermsattendance.json();
          } else {
              error = `Failed to fetch midtermsattendance: ${midtermsattendance.statusText}`;
          }


     
  });

  
  // Handle the form submission for adding a grade
  async function handleSubmit(event) {
      event.preventDefault();  // Prevent default form behavior

      const token = localStorage.getItem('jwtToken');
      try {
          const response = await fetch('http://localhost:4000/teacher/addattendance', {
              method: 'POST',
              headers: {
                  'Content-Type': 'application/json',
                  'Authorization': `Bearer ${token}` // Include JWT token
              },
              body: JSON.stringify({
                  classid,
                  term,
                  scoretype,
                  attendanceDate
              })
          });

          if (response.ok) {
              successMessage = 'Attendance Added Successfully';
              error = '';
              // Optionally clear the form or reset fields
              attendanceDate = '';
         

            // Optionally reload the page or fetch data again
            setTimeout(() => {
                  window.location.reload();
              }, 2000);  // 2 seconds delay
          } else {
              const result = await response.json();
              error = `${result.message}`;
              successMessage = '';
          }
      } catch (err) {
          error = `Error adding attendance: ${err.message}`;
          successMessage = '';
      }
  }

</script>

{#if classinfo}
<div class="mb-3">
    <!-- <a class="btn btn-sm btn-outline-secondary rounded-0"  href={`/Teacher/MyClass`}>
        ← go back to classlists 
      </a> -->
</div>

<!-- Card with Class Info -->
<!-- <div class="card rounded-0 shadow-sm mb-2">
  <div class="card-body"> -->
      <!-- Header of Class Info -->
      <!-- <h2 class="card-title mb-3">
          {classinfo.Subjectitle?.title} ({classinfo.subjectcode})
      </h2>
      <div class="d-flex justify-content-start gap-4 mb-1">
          <p class="text-muted mb-0"><strong>Year: </strong> <strong> {classinfo.year}</strong></p>
          <p class="text-muted mb-0"><strong>Semester: </strong><strong>{classinfo.semester}</strong></p>
          <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</strong></p>
      </div> -->

      <a class="btn btn-sm rounded-0 btn-primary  p-2 mb-3"  href={`/Teacher/MyClass`}>
        ←  go back to classlists 
      </a>

      

      <!-- Nav Tabs as Links -->
      <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
        <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 text-center bg-light" href={`/Teacher/${classid}`} role="tab" aria-selected="false">GRADES</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="false">PRELIM</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link active rounded-0 text-center" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
          </li>
      </ul>

      <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
          <li class="nav-item flex-grow-1">
              <a class="nav-link active disabled rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Attendance`} role="tab" aria-selected="false">ATTENDANCE</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Participation`} role="tab" aria-selected="false">PARTICIPATION</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Quiz`} role="tab" aria-selected="false">QUIZ</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Activity-Project`} role="tab" aria-selected="false">ACTIVITY/PROJECT</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Exam`} role="tab" aria-selected="false">EXAM</a>
          </li>
      </ul>

      <!-- Container Row -->
      <div class="row">
        <!-- Cards displaying attendance data (left side, scrollable) -->
        <div class="col-md-8" style="max-height: 50vh;">
            <div class="row mt-3">
                {#each attendancemidterm as attendancemidterm}
                <div class="col-md-3">
                    <div class="acard card rounded-0 text-center mb-3 shadow-sm">
                        <a href={`/Teacher/${classid}/Midterm/Attendance/${attendancemidterm.gradeid}`} style="text-decoration: none; color: inherit;">
                          <div class="card-body p-2 pb-4">
                            <div class="d-flex justify-content-between align-items-center">
                                <div>
                                    <p class="m-0 p-2">{attendancemidterm.scoretype}</p>
                                </div>
                                <div class="dropdown">
                                  <button class="btn dropdown-toggle" type="button" data-bs-toggle="dropdown" aria-expanded="false" >
                                
                                  </button>
                                  <ul class="dropdown-menu p-3">
                                      <li>
                                          <button class="text-decoration-none border border-0 bg-transparent" type="button"
                                          >
                                              Edit date
                                          </button>
                                      </li>
                                      <li>
                                          <button class="text-decoration-none border border-0 bg-transparent" type="button">
                                              Delete
                                          </button>
                                      </li>
                                  </ul>
                                </div>
                            </div>
                
                            <strong class="mt-2 pt-2"> {new Date(attendancemidterm.created).toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' })}</strong>
                        </div>
                      
                        </a>
                     
                    </div>
                </div>
                {/each}
            </div>
        </div>


          <!-- Form for adding attendance (right side, smaller and aligned) -->
          <div class="col-md-4">
              <form class="form-inline mt-3 ms-auto" on:submit|preventDefault={handleSubmit} style="width: 90%;">
                  <input type="hidden" name="classid" value={classid} />
                  <input type="hidden" name="term" value={term} />
                  <input type="hidden" name="scoretype" value={scoretype} />

                  <!-- AttendanceDate Input -->
                  <div class="form-group mb-2 ">
                      <label for="attendanceDate" class="me-2">Attendance Date:</label>
                      <input 
                          type="date" 
                          id="attendanceDate" 
                          class="form-control form-control-sm" 
                          bind:value={attendanceDate} 
                          style="width: 180px;" 
                      />
                  </div>

                  <!-- Submit Button -->
                  <div class="mt-4">
                      <button class="btn btn-primary btn-sm" type="submit">Add Attendance</button>
                  </div>

                  <!-- Success Message -->
                  {#if successMessage}
                  <div class="alert alert-success mt-3">{successMessage}</div>
                  {/if}

                  <!-- Error Message -->
                  {#if error}
                  <div class="alert alert-danger mt-3">{error}</div>
                  {/if}
              </form>
          </div>
      </div>
  <!-- </div>
</div> -->
{/if}




  <!-- Second Card with Nav Tabs as Links -->
  <!-- <div class="card rounded-0 shadow-sm mb-4">
      <div class="card-body">
      </div> 
  </div> -->
  
    





<style>
  .nav-link:hover {
      background-color: #001A56 !important; /* Bootstrap primary color or any custom color */
      color: rgb(255, 255, 255) !important; /* Make the text white when hovered */
      transition: background-color 0.2s ease-in-out !important; /* Smooth transition */
      
  }

  .acard:hover {
      border-color: #007bff !important; /* Blue border on hover */
  }
  .active {
        background-color: #001A56 !important; /* Make the background of navtab blue when active */
        color: rgb(255, 255, 255) !important; /* Make the text white when active */
    }
</style>