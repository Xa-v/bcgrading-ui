<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { jwtDecode } from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classinfo = [];
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

        await fetchClassAndAttendanceData();
    });

     // Fetch class and attendance data
     async function fetchClassAndAttendanceData() {
        const token = localStorage.getItem('jwtToken');
        const decodedToken = jwtDecode(token);
        userRole = decodedToken.role;  // Save userRole for later use
        try {
            // Fetch class details
            const classdetails = await fetch(`http://localhost:4000/registrar/classinfo/${classid}`, {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });
            if (classdetails.ok) {
                classinfo = await classdetails.json();
            } else {
                throw new Error('Failed to fetch class details');
            }

            // Fetch attendance scores
            const attendancescores = await fetch(`http://localhost:4000/teacher/getgradelist/${gradeid}`, {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });
            if (attendancescores.ok) {
                attendancescoresprelim = await attendancescores.json();
            } else {
                throw new Error('Failed to fetch attendance scores');
            }
        } catch (err) {
            error = err.message;
        }
    }

    // Function to handle updating attendanceStatus
    async function updateAttendanceStatus(scoreid, newStatus) {
        const token = localStorage.getItem('jwtToken');
        
        try {
            const response = await fetch(`http://localhost:4000/teacher/updateattendance/${scoreid}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({ attendanceStatus: newStatus })
            });

            if (response.ok) {
                const result = await response.json();
                console.log(result.message);
                await fetchClassAndAttendanceData();
            } else {
                console.error('Failed to update attendance status.');
            }
        } catch (err) {
            console.error('Error:', err);
        }
    }
</script>

{#if classinfo}
<div class="mb-3">
  
</div>
<div class="card rounded-0 shadow-sm mb-2">
    <div class="card-body">
        <h2 class="card-title mb-3">
            {classinfo.Subjectitle?.title} ({classinfo.subjectcode})
        </h2>
        <div class="d-flex justify-content-start gap-4 mb-1">
            <p class="text-muted mb-0"><strong>Year: </strong> {classinfo.year}</p>
            <p class="text-muted mb-0"><strong>Semester: </strong> {classinfo.semester}</p>
            <p class="text-muted"><strong>Teacher: </strong> {classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName}</p>
        </div>

          <a class="btn btn-sm rounded-0 btn-primary  p-2 mb-3"  href={`/Teacher/${1}/Prelim/Attendance`}>
            ←  go back to attendance 
          </a>

        <!-- Nav Tabs as Links -->
        <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 text-center bg-light" href={`/Teacher/${classid}`} role="tab" aria-selected="false">GRADES</a>
              </li>
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

        <div class="d-flex align-items-center">
            <p class="mb-0 ms-3"> <strong>PRESENT = 10 | LATE = 7 | EXCUSED = 5 | ABSENT KAY = 0 F***YOU MGA SIG ABSENT </strong>  </p>
        </div>
        
        <!-- Table displaying attendance data with editable attendanceStatus -->
        <div style="max-height: 42vh; overflow-y: auto; overflow-x: hidden;">
            <table class="table table-bordered table-hover">
                <thead class="table-light">
                    <tr>
                        <!-- <th scope="col">Score ID</th>
                        <th scope="col">Grade ID</th> -->
                        <th scope="col">Name</th>
                        <th scope="col">Attendance Status</th>
                        <th scope="col">Score</th>
                        <th scope="col">Perfect Score</th>
                     
                    </tr>
                </thead>
                <tbody>
                    {#each attendancescoresprelim as attendance}
                    <tr>
                        <!-- <td>{attendance.scoreid}</td>
                        <td>{attendance.gradeid}</td> -->
              
                        <td>{attendance.Studentlist?.studentinfo?.lastName}{attendance.Studentlist?.studentinfo?.firstName}</td>

                        <td>
                            <select
                                class="form-select border rounded-0"
                                class:border-danger="{attendance.attendanceStatus === ''}"
                                class:border-success="{attendance.attendanceStatus !== ''}"
                                bind:value={attendance.attendanceStatus}
                                on:change={(e) => updateAttendanceStatus(attendance.scoreid, e.target.value)}
                            >
                                <option value="Present">Present</option>
                                <option value="Absent">Absent</option>
                                <option value="Late">Late</option>
                                <option value="Excused">Excused</option>
                            </select>
                        </td>
                        
                        <td>{attendance.score}</td>
                        <td>{attendance.perfectscore}</td>
                    </tr>
                    {/each}
                </tbody>
            </table>
        </div>
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
        background-color: #001A56 !important; 
        color: rgb(255, 255, 255) !important; 
        transition: background-color 0.2s ease-in-out !important; 
    }
    .active {
        background-color: #001A56 !important; 
        color: rgb(255, 255, 255) !important;
    }
</style>



  <!-- //comment para mo highlight -->