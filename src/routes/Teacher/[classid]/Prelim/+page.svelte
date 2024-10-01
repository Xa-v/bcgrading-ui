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

      // Fetch class details
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
      const gradecomputed = await fetch(`http://localhost:4000/teacher/gradesofstudentsprelim/${classid}`, {
          headers: {
              'Authorization': `Bearer ${token}` // Include JWT token
          }
      });

      if (gradecomputed.ok) {
          const data = await gradecomputed.json();
          students = data.students;  // Update students array
      } else {
          error = `Failed to fetch gradecomputed: ${gradecomputed.statusText}`;
      }
  });
</script>

<!-- Display class info and navigation links -->
{#if classinfo}
<div class="mb-3">
   
  </div>
  <!-- Card with Class Info -->
  <!-- <div class="card rounded-0 shadow-sm mb-2">
      <div class="card-body">
          <h2 class="card-title mb-3">
              {classinfo.Subjectitle?.title} ({classinfo.subjectcode})
          </h2>
          <div class="d-flex justify-content-start gap-4 mb-1">
              <p class="text-muted mb-0"><strong>Year: </strong><strong>{classinfo.year}</strong></p>
              <p class="text-muted mb-0"><strong>Semester: </strong><strong>{classinfo.semester}</strong></p>
              <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</strong></p>
          </div> -->

          <a class="btn btn-sm rounded-0 btn-primary  p-2 mb-3"  href={`/Teacher/MyClass`}>
            ←  go back to classlists 
          </a>

          <!-- Navigation Links -->
          <ul class="nav nav-tabs rounded-0" id="myTab" role="tablist">
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 text-center bg-light" href={`/Teacher/${classid}`} role="tab" aria-selected="false">GRADES</a>
              </li>
              <li class="nav-item flex-grow-1">
                  <a class="nav-link active rounded-0 text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="true">PRELIM</a>
              </li>
              <li class="nav-item flex-grow-1">
                  <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
              </li>
              <li class="nav-item flex-grow-1">
                  <a class="nav-link rounded-0 text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
              </li>
          </ul>

          <!-- Sub Navigation Links -->
          <ul class="nav nav-tabs rounded-0" id="myTab" role="tablist">
          
              <li class="nav-item flex-grow-1">
                  <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Attendance`} role="tab" aria-selected="true">ATTENDANCE</a>
              </li>
              <li class="nav-item flex-grow-1">
                  <a class="nav-link rounded-0 border text-center" href={`/Teacher/${classid}/Prelim/Participation`} role="tab" aria-selected="false">PARTICIPATION</a>
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

          
      <div style="max-height: 45vh; overflow-y: auto; overflow-x: hidden;">
        {#if students && students.length > 0}
            <!-- Students Table -->
            <table class="table table-bordered table-hover">
                <thead class="table-light">
                    <tr>
                        <th>Name</th>
                  
                        <th>Term</th>
                        <!-- <th>Total Attendance Score</th>
                        <th>Perfect Attendance Score</th>
                        <th>Attendance 5%</th> -->
                        <!-- <th>Total Participation Score</th>
                        <th>Perfect Participation Score</th>
                        <th>Participation 5%</th>
                        <th>Total Quiz Score</th>
                        <th>Perfect Quiz Score</th>
                        <th>Quiz 15%</th>
                        <th>Total Activity-Project Score</th>
                        <th>Perfect Activity-Project Score</th>
                        <th>Activity-Project 45%</th>
                        <th>Total Exam Score</th>
                        <th>Perfect Exam Score</th>
                        <th>Exam 45%</th> -->
                        <th>Final Grade</th>
                        <th>Transmuted Grade</th>
                    </tr>
                </thead>
                <tbody>
                    {#each students as student}
                        {#each student.ComputedGradelists as grade}
                            <tr>
                                <td>{student.studentinfo.lastName} {student.studentinfo.firstName}</td>
                             
                                <td>{grade.term}</td>
                                <!-- <td>{grade.totalattendance}</td>
                                <td>{grade.perfectattendancescore}</td>
                                <td>{grade.attendance5percent}</td> -->
                                <!-- <td>{grade.totalparticipation}</td>
                                <td>{grade.perfectparticipationscore}</td>
                                <td>{grade.participation5percent}</td>
                                <td>{grade.totalquiz}</td>
                                <td>{grade.perfectquizscore}</td>
                                <td>{grade.quiz15percent}</td>
                                <td>{grade.totalactivityproject}</td>
                                <td>{grade.perfectactivityprojectscore}</td>
                                <td>{grade.activityproject45percent}</td>
                                <td>{grade.totalexam}</td>
                                <td>{grade.perfectexamscore}</td>
                                <td>{grade.exam30percent}</td> -->
                                <td>{grade.finalcomputedgrade}</td>
                                <td>{grade.transmutedgrade}</td>
                            </tr>
                        {/each}
                    {/each}
                </tbody>
            </table>
        {:else}
        <div class="alert alert mt-3">No grades/scores available yet please add scores</div>
        <table class="table table-bordered table-hover">
            <thead class="table-light">
                <tr>
                    <th>Name</th>
                    <th>Term</th>   
                    <th>Final Grade</th>
                    <th>Transmuted Grade</th>
                </tr>
            </thead>
            <tbody>
                {#each students as student}
                    {#each student.ComputedGradelists as grade}
                        <tr>
                            <td>{student.studentinfo.lastName} {student.studentinfo.firstName}</td>
                         
                            <td>{grade.term}</td>
                   
                            <td>-</td>
                            <td>-</td>
                        </tr>
                    {/each}
                {/each}
            </tbody>
        </table>
        {/if}
    </div>

      <!-- </div>
  </div> -->


{/if}





<!-- Display any error messages -->
<!-- {#if error}
<div class="alert alert-danger mt-4">
  {error}
</div>
{/if} -->

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
