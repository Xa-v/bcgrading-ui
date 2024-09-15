<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';

    let classinfo = [];
    let students = [];
    let error = '';
    let userRole = '';
    
    $: ({ classid } = $page.params);

    onMount(async () => {
        await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');
        const decodedToken = jwtDecode(token);
        userRole = decodedToken.role;  // Save userRole for later use

        // Fetch the class details
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

        // Fetch the students list
        const studentlist = await fetch(`http://localhost:4000/registrar/studentlist/${classid}`, {
            headers: {
                'Authorization': `Bearer ${token}` // Include JWT token
            }
        });

        if (studentlist.ok) {
            const studentData = await studentlist.json();
            // Check if the data is properly structured and log it for debugging
            // console.log("Fetched students:", studentData);
            students = studentData.students; // Ensure correct path to student data
        } else {
            error = `Failed to fetch studentlist: ${studentlist.statusText}`;
        }
    });
</script>

<!-- Make sure classinfo is available before rendering navigation links -->
{#if classinfo}
    <div class="mb-3">
        <a class="btn btn-sm btn-outline-secondary" href="/Registrar/ClassList">
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

        <ul class="nav nav-tabs rounded-0 " id="myTab" role="tablist">
            <li class="nav-item">
              <a class="nav-link active disabled rounded-0 " href={`/Registrar/${classinfo.classid}/StudentList`} role="tab" aria-selected="true">Students List</a>
            </li>
            <li class="nav-item">
              <a class="nav-link rounded-0" href={`/Registrar/${classinfo.classid}/AddStudent`} role="tab" aria-selected="false">Add Student</a>
            </li>
            <li class="nav-item">
              <a class="nav-link  rounded-0" href={`/Registrar/${classinfo.classid}`} role="tab" aria-selected="false">Scores</a>
            </li>
          </ul>
      </div>
    </div>
{/if}

<!-- Table for Student Info -->
{#if students && Array.isArray(students) && students.length > 0}
    <div class="card rounded-0 shadow-sm mb-4">
        <div class="card-body">
            <!-- Display Students List Inside a Bordered Bootstrap Table -->
            <table class="table w-auto m-0 table-bordered border-dark">
                <thead>
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
                            <td>{student.studentinfo?.lastName}</td>
                            <td>{student.studentinfo?.firstName}</td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        </div>
    </div>
{:else} 
<div class="card rounded-0 shadow-sm mb-4">
    <div class="card-body">
    <h1 class="alert  mt-3">No students available</h1>
</div>
</div>
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
        background-color: #001A56 !important; /* Bootstrap primary color or any custom color */
        color: rgb(255, 255, 255) !important; /* Make the text white when hovered */
  
        
    }
  </style>