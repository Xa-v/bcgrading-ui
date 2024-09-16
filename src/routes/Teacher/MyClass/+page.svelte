<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';
    let years = [];
    let semesters = [];
    let subjects = [];
    let classes = [];
    let error = '';
    let userRole = '';
    let userID = '';
   
 $: ({ classid } = $page.params);

    onMount(async () => {

       // Import Bootstrap JS
       await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

        
      
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;  
            userID = decodedToken.id;
            console.log("ID:", userID);

            // Fetch the years if the user is authorized
            const yearlist = await fetch(`http://localhost:4000/teacher/years/${userID}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (yearlist.ok) {
                years = await yearlist.json();
            } else {
                error = `Failed to fetch years: ${yearlist.statusText}`;
            }


            const semesterlist = await fetch(`http://localhost:4000/teacher/semesters/${userID}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (semesterlist.ok) {
                semesters = await semesterlist.json();
            } else {
                error = `Failed to fetch semesters: ${semesterlist.statusText}`;
            }
            

            const subjectlist = await fetch(`http://localhost:4000/teacher/subjects/${userID}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (subjectlist.ok) {
                subjects = await subjectlist.json();
            } else {
                error = `Failed to fetch subjects: ${subjectlist.statusText}`;
            }

            
            const classlist = await fetch(`http://localhost:4000/teacher/${userID}`, {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (classlist.ok) {
                classes = await classlist.json();
            } else {
                error = `Failed to fetch classLIST: ${classlist.statusText}`;
            }
       
    });

    let filteredClasses = [];
   
    let selectedYear = '';
    let selectedSemester = '';
    let selectedSubject = '';

    function filterClasses() {
    console.log('Selected Year:', selectedYear);
    console.log('Selected Semester:', selectedSemester);
    console.log('Selected Subject:', selectedSubject);

    if (selectedYear === '' && selectedSemester === '' && selectedSubject === '') {
        filteredClasses = [...classes];
    } else {
        filteredClasses = classes.filter(cls => {
            return (
                (selectedYear === '' || cls.year === selectedYear) &&
                (selectedSemester === '' || cls.semester === selectedSemester) &&
                (selectedSubject === '' || cls.subjectcode === selectedSubject)
            );
        });
    }

    console.log('Filtered Classes:', filteredClasses);
}


    function handleYearSelect(year) {
        console.log('Year selected:', year);
        selectedYear = year;
        filterClasses();
    }

    function handleSemesterSelect(semester) {
        console.log('Semester selected:', semester);
        selectedSemester = semester;
        filterClasses();
    }

    function handleSubjectSelect(subject) {
        console.log('Subject selected:', subject);
        selectedSubject = subject;
        filterClasses();
    }



    function clearFilters() {
    selectedYear = '';
    selectedSemester = '';
    selectedSubject = '';
    filterClasses();
}



</script>

<div class="container text-center">
    <div class="btn-group me-2">
        <button type="button" style="width: 160px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown" aria-expanded="false">
        YEAR:  {selectedYear}            
        </button>
        <ul class="dropdown-menu" style="max-height: 200px; overflow-y: auto;">
            {#each years as year}
            <li class="bg-light border"><button class="dropdown-item" on:click={() => handleYearSelect(year.year)}>{year.year}</button></li>
            {/each}
        </ul>
    </div>

    <div class="btn-group me-2">
        <button type="button" style="width: 190px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown">
        SEMESTER:  {selectedSemester}          
        </button>
        <ul class="dropdown-menu" style="max-height: 200px; overflow-y: auto; width: 190px !important;">
            {#each semesters as semester}
            <li class="bg-light border">
                <button class="dropdown-item" on:click={() => handleSemesterSelect(semester.semester)}>{semester.semester}</button>
            </li>
            {/each}
        </ul>
    </div>

    <div class="btn-group">
        <button type="button" style="width: 500px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown">
         SUBJECT: {selectedSubject}           
        </button>
        <ul class="dropdown-menu" style="max-height: 200px; overflow-x: hidden; width: 500px !important;">
            {#each subjects as subject}
            <li class="bg-light border">
                <button class="dropdown-item" on:click={() => handleSubjectSelect(subject.subjectcode)}>{subject.subjectcode} - {subject.Subjectitle.title}</button>
            </li>
            {/each}
        </ul>
    </div>
    <button type="button" class="btn text-white btn-dark" on:click={clearFilters}>Clear Filters <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16">
        <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/>
      </svg></button>
</div>



    





{#if classes.length > 0}
    <h3 class="m-2 py-4">Teacher:  {classes[0].TeacherInfo?.firstName} {classes[0].TeacherInfo?.lastName}</h3>
{/if}



<!-- {#if !error} -->
{#if !error}
    {#if selectedYear === '' && selectedSemester === '' && selectedSubject === ''}
        <!-- Show entire table if no filters are selected -->
       
      
        <div style="max-height: 73vh; overflow-y: auto;">
        <table class="table table-bordered" style="width: 100% !important;">
            <thead>
                <tr>
                    <th>Grades</th>
                    <th>Subject Code</th>
                    <th>Subject Title</th>
                    <th>Class ID</th>
                    <th>Year</th>
                    <th>Semester</th>
                    <!-- <th>Teacher ID</th>
                    <th>Created</th>
                    <th>Updated</th> -->
                    <th>Status</th>
                   
                </tr>
            </thead>
            <tbody>
                {#each classes as classlist}
                <tr>
                    <td><a class="btn btn-outline-primary rounded-0 w-100" href={`/Teacher/${classlist.classid}`}>SELECT</a></td>                 
                    <td>{classlist.subjectcode}</td>
                    <td>{classlist.Subjectitle.title}</td>
                    <td>{classlist.classid}</td>
                    <td>{classlist.year}</td>
                    <td>{classlist.semester}</td>
                    <!-- <td>{classlist.teacherid}</td>
                    <td>{classlist.created}</td>
                    <td>{classlist.updated}</td> -->
                    <td>
                        {#if classlist.isActive}
                            <p class="badge-lg text-center text-bg-success">Active</p>
                        {:else}
                            <p class="badge-lg text-center text-bg-danger">Inactive</p>
                        {/if}
                    </td>
                  
                </tr>
                {/each}
            </tbody>
        </table>
        </div>
    {:else}
        {#if filteredClasses.length > 0}
            <!-- Show filtered table if there are results -->
            <div style="max-height: 73vh; overflow-y: auto;">
                <table class="table table-bordered" style="width: 100% !important;">
                <thead>
                    <tr>
                        <th>Grades</th>
                        <th>Subject Code</th>
                        <th>Subject Title</th>
                        <th>Class ID</th>
                        <th>Year</th>
                        <th>Semester</th>
                        <!-- <th>Teacher ID</th>
                        <th>Created</th>
                        <th>Updated</th> -->
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody>
                    {#each filteredClasses as classlist}
                    <tr>
                        <td><a href={`/Teacher/${classlist.classid}`}>SELECT</a></td>                 
                        <td>{classlist.subjectcode}</td>
                        <td>{classlist.Subjectitle.title}</td>
                        <td>{classlist.classid}</td>
                        <td>{classlist.year}</td>
                        <td>{classlist.semester}</td>
                        <!-- <td>{classlist.teacherid}</td>
                        <td>{classlist.created}</td>
                        <td>{classlist.updated}</td> -->
                        <td>
                            {#if classlist.isActive}
                                <p class="badge-lg text-center text-bg-success">Active</p>
                            {:else}
                                <p class="badge-lg text-center text-bg-danger">Inactive</p>
                            {/if}
                        </td>
                    </tr>
                    {/each}
                </tbody>
            </table>
            </div>
        {:else}
            <h1 class="text-center">No classes found </h1>
        {/if}
    {/if}
{/if}



{#if error}
    <p>{error}</p>
{/if}


