<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
    let attendanceprelim = [];
    let error = '';
    let successMessage = '';
    let userRole = '';

    let term = 'Prelim';  // Default to Prelim
    let scoretype = 'Attendance';  // Default to Attendance
    let attendanceDate = '';

    
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

            const prelimsattendance = await fetch(`http://localhost:4000/teacher/Prelim/Attendance/${classid}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (prelimsattendance.ok) {
                attendanceprelim = await prelimsattendance.json();
            } else {
                error = `Failed to fetch prelimsattendance: ${prelimsattendance.statusText}`;
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
                error = `Failed to add attendance: ${result.message}`;
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
    <a class="btn btn-sm btn-outline-secondary" href={`/Teacher/MyClass`}>
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
            <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</strong></p>
        </div>

        <!-- Nav Tabs as Links -->
        <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
            <li class="nav-item flex-grow-1">
                <a class="nav-link active disabled rounded-0 text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="true">PRELIM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
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
                <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Quiz`} role="tab" aria-selected="false">QUIZ</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Activity-Project`} role="tab" aria-selected="false">ACTIVITY/PROJECT</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Exam`} role="tab" aria-selected="false">EXAM</a>
            </li>
        </ul>

        <!-- Container Row -->
        <div class="row">
            <!-- Cards displaying attendance data (left side, scrollable) -->
            <div class="col-md-8" style="max-height: 50vh; overflow-y: auto;">
                <div class="row mt-3">
                    {#each attendanceprelim as attendance}
                    <div class="col-md-4">
                        <a href={`/Teacher/${classid}/Prelim/Attendance/${attendance.gradeid}`} style="text-decoration: none; color: inherit;">
                        <div class="card rounded-0 text-center mb-3 shadow-sm border acard" style="width: 15rem; height: 7rem;">
                            
                                <div class="card-body p-2">
                                    <!-- <p class="card-title">{attendance.scoretype}</p> -->
                                    <p class="card-text">Attendance DATE:</p>
                                  
                                    <strong>{new Date(attendance.attendanceDate).toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' })}</strong>
                                </div>
                        </div>
                    </a>  
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
    </div>
</div>
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
  </style>