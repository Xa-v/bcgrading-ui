<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
    let students = [];
    let selectedStudents = [];
    let error = '';
    let successMessage = '';
    let userRole = '';

    $: ({ classid } = $page.params);  // Get classid from route

    onMount(async () => {
       // Import Bootstrap JS
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
            error = `Failed to fetch class details: ${classdetails.statusText}`;
        }

        // Fetch the students not in the class
        const studentList = await fetch(`http://localhost:4000/registrar/studentnotinclass/${classid}`, {
            headers: {
                'Authorization': `Bearer ${token}` // Include JWT token
            }
        });

        if (studentList.ok) {
            students = await studentList.json();
        } else {
            error = `Failed to fetch students not in class: ${studentList.statusText}`;
        }
    });

    // Function to handle checkbox changes
    function toggleStudentSelection(studentId) {
        if (selectedStudents.includes(studentId)) {
            selectedStudents = selectedStudents.filter(id => id !== studentId);
        } else {
            selectedStudents = [...selectedStudents, studentId];
        }
    }

    async function addStudents(event) {
    event.preventDefault();  // Prevent form submission default behavior

    const token = localStorage.getItem('jwtToken');

    try {
        let successCount = 0; // Track the number of successful additions
        let failedCount = 0; // Track the number of failed additions
        const failures = [];

        // Loop through each selected student and send a separate POST request
        for (let studentId of selectedStudents) {
            const response = await fetch('http://localhost:4000/registrar/addstudentToclass', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({
                    classid: classid, // Pass classid from route
                    studentid: studentId // Send each student ID individually
                })
            });

            const data = await response.json();
            if (response.ok) {
                successCount += 1;
            } else {
                failedCount += 1;
                failures.push(studentId); // Track which student failed
                console.error(`Failed to add student ID: ${studentId}`, data.message);
            }
        }

        // Build the success message
        if (successCount > 0) {
            successMessage = `${successCount} students added successfully`;

            // Clear selected students
            selectedStudents = [];

            // Refresh the page after a short delay (e.g., 1.5 seconds)
            setTimeout(() => {
                goto(`/Registrar/${classid}/StudentList`);
            }, 1000);
        }

        // Handle failures
        if (failedCount > 0) {
            error = `${failedCount} students failed to be added: ${failures.join(', ')}`;
        }
        
    } catch (err) {
        error = `Error adding students: ${err.message}`;
    }
}

</script>

{#if classinfo}
<div class="mb-3">
    <button class="btn btn-sm btn-outline-secondary" on:click={() => window.history.back()}>
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
            <p class="text-muted mb-0"><strong>Year: </strong> <strong>{classinfo.year}</strong></p>
            <p class="text-muted mb-0"><strong>Semester: </strong><strong>{classinfo.semester}</strong></p>
            <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</strong></p>
        </div>

    <!-- Nav Tabs as Links -->
    <ul class="nav nav-tabs rounded-0" id="myTab" role="tablist">
        <li class="nav-item">
            <a class="nav-link rounded-0" href={`/Registrar/${classinfo.classid}/StudentList`} role="tab" aria-selected="true">Students List</a>
        </li>
        <li class="nav-item">
            <a class="nav-link  rounded-0 active" href={`/Registrar/${classinfo.classid}/AddStudent`} role="tab" aria-selected="false">Add Student</a>
        </li>
        <li class="nav-item">
            <a class="nav-link rounded-0" href={`/Registrar/${classinfo.classid}`} role="tab" aria-selected="false">Scores</a>
        </li>
    </ul>

    <!-- Add Student Form -->
    <form on:submit={addStudents}>
        <input type="hidden" name="classid" value={classid} />  <!-- Hidden classid field -->
   
        <div class="mb-3">
            <h1 class="text-center">Select Students to Add</h1>
         
            <!-- Bootstrap Table -->
            <table class="table table-bordered table-hover">
                <thead class="table-light">
                    <tr>
                        <th scope="col">Select</th>
                        <th scope="col">First Name</th>
                        <th scope="col">Last Name</th>
                        <th scope="col">Student ID</th>
                    </tr>
                </thead>
                <tbody>
                    {#each students as student}
                    <tr>
                        <td>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="student-{student.id}" value={student.id} on:change={() => toggleStudentSelection(student.id)}>
                            </div>
                        </td>
                        <td>{student.firstName}</td>
                        <td>{student.lastName}</td>
                        <td>{student.id}</td>
                    </tr>
                    {/each}
                </tbody>
            </table>
        </div>
        <button type="submit" class="btn btn-primary">Add Selected Students</button> 

   
    </form>

    <!-- Success or Error Messages -->
    {#if successMessage}
        <div class="alert alert-success mt-3">{successMessage}</div>
    {/if}

    {#if error}
        <div class="alert alert-danger mt-3">{error}</div>
    {/if}

    </div>
</div>

<!-- Second Card with Nav Tab-->
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
        background-color: #001A56 !important; /* Bootstrap primary color or any custom color */
        color: rgb(255, 255, 255) !important; /* Make the text white when hovered */
  
        
    }
  </style>