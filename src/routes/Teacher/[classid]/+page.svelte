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
    sessionStorage.clear();
      // Import Bootstrap JS
      await import('bootstrap/dist/js/bootstrap.bundle.min.js');

      const token = localStorage.getItem('jwtToken');
      const decodedToken = jwtDecode(token);
      userRole = decodedToken.role;  // Save userRole for later use

      // Fetch class details
      const classdetails = await fetch(`http://localhost:4000/registrar/classinfo/${classid}`, {
          headers: { 'Authorization': `Bearer ${token}` }
      });

      if (classdetails.ok) {
          classinfo = await classdetails.json();
      } else {
          error = `Failed to fetch classdetails: ${classdetails.statusText}`;
      }

      // Fetch grades for Prelim, Midterm, and Final
      let prelim = await fetchGradeData(`http://localhost:4000/teacher/gradesofstudentsprelim/${classid}`, token);
      let midterm = await fetchGradeData(`http://localhost:4000/teacher/gradesofstudentsmidterm/${classid}`, token);
      let final = await fetchGradeData(`http://localhost:4000/teacher/gradesofstudentsfinal/${classid}`, token);

      // Merge grades by studentid
      mergeGrades(prelim, midterm, final);
  });

  // Function to fetch grade data from API
  async function fetchGradeData(url, token) {
      const res = await fetch(url, {
          headers: { 'Authorization': `Bearer ${token}` }
      });

      if (res.ok) {
          const data = await res.json();
          return data.students; // Return students array from response
      } else {
          error = `Failed to fetch data from ${url}`;
          return [];
      }
  }

  // Function to merge grades by studentid
  function mergeGrades(prelim, midterm, final) {
      students = prelim.map(student => {
          const midtermStudent = midterm.find(m => m.studentid === student.studentid) || {};
          const finalStudent = final.find(f => f.studentid === student.studentid) || {};

          return {
              studentinfo: student.studentinfo,
              grades: {
                  Prelim: student.ComputedGradelists[0]?.transmutedgrade || '5',
                  Midterm: midtermStudent.ComputedGradelists ? midtermStudent.ComputedGradelists[0]?.transmutedgrade : '5',
                  Final: finalStudent.ComputedGradelists ? finalStudent.ComputedGradelists[0]?.transmutedgrade : '5'
              }
          };
      });
  }
</script>

{#if classinfo}
<div class="mb-3">
</div>

<!-- Card with Class Info -->
<!-- <div class="card rounded-0 shadow-sm mb-2">
  <div class="card-body">
    <h2 class="card-title mb-3">{classinfo.Subjectitle?.title} ({classinfo.subjectcode})</h2>
    <div class="d-flex justify-content-start gap-4 mb-1">
      <p class="text-muted mb-0"><strong>Year:</strong> {classinfo.year}</p>
      <p class="text-muted mb-0"><strong>Semester:</strong> {classinfo.semester}</p>
      <p class="text-muted"><strong>Teacher:</strong> {classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</p>
    </div> -->

    <a class="btn btn-sm rounded-0 btn-primary p-2 mb-3" href={`/Teacher/MyClass`}>
      ← go back to classlists
    </a>

    <ul class="nav nav-tabs rounded-0" id="myTab" role="tablist">
      <li class="nav-item flex-grow-1">
          <a class="nav-link active disabled rounded-0 text-center bg-light" href={`/Teacher/${classid}`} role="tab" aria-selected="false">GRADES</a>
        </li>
        <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="true">PRELIM</a>
        </li>
        <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
        </li>
        <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
        </li>
    </ul>

<!-- Grades Table -->
<div style="max-height: 45vh; overflow-y: auto; overflow-x: hidden;">
  {#if students.length > 0}
  <table class="table table-bordered table-hover">
    <thead class="table-light">
      <tr>
        <th>Name</th>
        <th>Prelim Grade</th>
        <th>Midterm Grade</th>
        <th>Final Grade</th>
      </tr>
    </thead>
    <tbody>
      {#each students as student}
      <tr>
        <td>{student.studentinfo.lastName}, {student.studentinfo.firstName}</td>
        <td>{student.grades.Prelim}</td>
        <td>{student.grades.Midterm}</td>
        <td>{student.grades.Final}</td>
      </tr>
      {/each}
    </tbody>
  </table>
  {:else}
  <div class="">No students found.</div>
  {/if}
</div>
  <!-- </div>
</div> -->
{/if}

<!-- {#if error}
<div class="alert alert-danger mt-4">{error}</div>
{/if} -->

<style>
  .nav-link:hover {
    background-color: #001A56 !important;
    color: rgb(255, 255, 255) !important;
    transition: background-color 0.2s ease-in-out !important;
  }
  .active {
    background-color: #001A56 !important;
    color: rgb(255, 255, 255) !important;
  }
</style>
