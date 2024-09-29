<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { jwtDecode } from 'jwt-decode';

    let years = [];
    let semesters = [];
    let subjects = [];
    let teachers = [];
    let error = '';
    let successMessage = '';
    let userRole = '';

    let selectedYear = '';
    let selectedSemester = '';
    let selectedSubject = '';
    let teacherId = ''; 


    onMount(async () => {
        sessionStorage.clear();
        await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');
         

            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;


            const yearlist = await fetch('http://localhost:4000/registrar/years', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (yearlist.ok) {
                years = await yearlist.json();
            } else {
                error = `Failed to fetch years: ${yearlist.statusText}`;
            }

            const semesterlist = await fetch('http://localhost:4000/registrar/semesters', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (semesterlist.ok) {
                semesters = await semesterlist.json();
            } else {
                error = `Failed to fetch semesters: ${semesterlist.statusText}`;
            }

            const subjectlist = await fetch('http://localhost:4000/registrar/subjects', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (subjectlist.ok) {
                subjects = await subjectlist.json();
            } else {
                error = `Failed to fetch subjects: ${subjectlist.statusText}`;
            }

            const teacherlist = await fetch('http://localhost:4000/registrar/teacherlist', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (teacherlist.ok) {
                teachers = await teacherlist.json();
            } else {
                error = `Failed to fetch teachers: ${teacherlist.statusText}`;
            }

    });

    async function addClass() {
        const token = localStorage.getItem('jwtToken');
        try {
            const response = await fetch('http://localhost:4000/registrar/addclass', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({
                    subjectcode: selectedSubject,
                    semester: selectedSemester,
                    year: selectedYear,
                    teacherid: teacherId
                })
            });

            if (response.ok) {
                successMessage = `Class added successfully!
                [SUBJECT: ${selectedSubject}] [Year: ${selectedYear}] [Semester: ${selectedSemester}] `;
                // [Teacher ID: ${teacherId}]
                // Hide the success message after 5 seconds
                setTimeout(() => {
                    successMessage = '';
                }, 5000);

                // Optionally, you can reset the selection
                selectedYear = '';
                selectedSemester = '';
                selectedSubject = '';
                teacherId = '';
                
            } else {
                error = `Failed to add class: ${response.statusText}`;
                setTimeout(() => {
                    error = '';
                }, 4000);
            }
        } catch (err) {
            error = `Error adding class: ${err.message}`;
        }
    }


</script>

<div class="container mt-4" style="height: 70vh;">
    <h2>Add Class</h2>

    {#if successMessage}
        <div class="alert alert-success text-bold" role="alert">
            {successMessage}
        </div>
    {/if}

    {#if error}
        <div class="alert alert-danger" role="alert">
            {error}
        </div>
    {/if}

    <div class="mb-3">
        <label for="subjectSelect" class="form-label">Select Subject</label>
        <div class="dropdown">
            <select id="subjectSelect" class="form-select" bind:value={selectedSubject} required size="1" style="max-height: 150px; overflow-y: auto;">
                <option value="">Choose...</option>
                {#each subjects as subject}
                    <option value={subject.subjectcode}>{subject.subjectcode} - {subject.title}</option>
                {/each}
            </select>
        </div>
    </div>
    
    <div class="mb-3">
        <label for="yearSelect" class="form-label">Select Year</label>
        <select id="yearSelect" class="form-select" bind:value={selectedYear} required>
            <option value="">Choose...</option>
            {#each years as year}
                <option value={year.year}>{year.year}</option>
            {/each}
        </select>
    </div>
    
    <div class="mb-3">
        <label for="semesterSelect" class="form-label">Select Semester</label>
        <select id="semesterSelect" class="form-select" bind:value={selectedSemester} required>
            <option value="">Choose...</option>
            {#each semesters as semester}
                <option value={semester.semester}>{semester.semester}</option>
            {/each}
        </select>
    </div>
    
    <!-- <div class="mb-3">
        <label for="teacherId" class="form-label">Teacher ID</label>
        <input type="text" id="teacherId" class="form-control" bind:value={teacherId} required placeholder="Enter Teacher ID" />
    </div> -->
    
    <div class="mb-3">
        <label for="teacherId" class="form-label">Select Teacher</label>
        <div class="dropdown">
            <select id="teacherId" class="form-select" bind:value={teacherId} required size="1" style="max-height: 150px; overflow-y: auto;">
                <option value="">Choose...</option>
                {#each teachers as teacher}
                    <option value={teacher.id} >{teacher.id} {teacher.firstName} {teacher.lastName}</option>
                {/each}
            </select>
        </div>
    </div>

    <button class="btn btn-primary" on:click={addClass}>Add Class</button>
</div>

<style>
    .container {
        max-width: 600px;
        max-height: 100vh;
    }
</style>
