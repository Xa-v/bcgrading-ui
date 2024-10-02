<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import {jwtDecode} from 'jwt-decode';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';
    import toast, {Toaster} from "svelte-french-toast";
    let classinfo = [];
    let participationscoresmidterm = [];
    let error = '';
    let userRole = '';
    let scoresToUpdate = {};
    $: ({ classid, gradeid } = $page.params);

    onMount(async () => {
        sessionStorage.clear();
       // Import Bootstrap JS
       await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

        
      
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;  // Save userRole for later use

            await fetchClassAndScoreData();
       
    });

  // Fetch class and score data with toast promise
  async function fetchClassAndScoreData() {
        const token = localStorage.getItem('jwtToken');
        const decodedToken = jwtDecode(token);
        userRole = decodedToken.role;

        const fetchPromise = (async () => {
            try {
                const classdetails = await fetch(`http://localhost:4000/registrar/classinfo/${classid}`, {
                    headers: {
                        'Authorization': `Bearer ${token}` // Include JWT token
                    }
                });

                if (classdetails.ok) {
                    classinfo = await classdetails.json();
                } else {
                    throw new Error(`Failed to fetch class details: ${classdetails.statusText}`);
                }

                const participation = await fetch(`http://localhost:4000/teacher/getgradelist/${gradeid}`, {
                    headers: {
                        'Authorization': `Bearer ${token}` // Include JWT token
                    }
                });

                if (participation.ok) {
                    participationscoresmidterm = await participation.json();
                } else {
                    throw new Error(`Failed to fetch activity project scores: ${participation.statusText}`);
                }
            } catch (err) {
                error = err.message;
                throw err;  // rethrow to pass error to toast
            }
        })();

  
    }

    // Update score with toast promise
    async function updateScore() {
        const token = localStorage.getItem('jwtToken');

        const updatePromise = (async () => {
            try {
                for (const [scoreid, newScore] of Object.entries(scoresToUpdate)) {
                    const response = await fetch(`http://localhost:4000/teacher/updatescore/${scoreid}`, {
                        method: 'PUT',
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': `Bearer ${token}`
                        },
                        body: JSON.stringify({ score: newScore })
                    });

                    if (!response.ok) {
                        throw new Error(`Failed to update Score for scoreid: ${scoreid}`);
                    }
                    const result = await response.json();
                    console.log(result.message);
                }
                await fetchClassAndScoreData();  // Refresh the data after update
            } catch (err) {
                console.error('Error:', err);
                throw err;  // rethrow to pass error to toast
            }
        })();

        toast.promise(
            updatePromise,
            {
                loading: "Updating scores...",
                success: "Scores updated successfully!",
                error: "Failed to update scores.",
            },
            { position: 'center-top' }
        );
    }

    function handleInputChange(scoreid, newScore) {
        scoresToUpdate[scoreid] = newScore;  // Store the updated score
    }

</script>
<Toaster/>

{#if classinfo}
<div class="mb-3">
 
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
            <p class="text-muted"><strong>Teacher: </strong><strong>{classinfo.TeacherInfo?.firstName} {classinfo.TeacherInfo?.lastName} </strong> </p>
          </div> -->

          <a class="btn btn-sm rounded-0 btn-primary  p-2 mb-3"  href={`/Teacher/${1}/Midterm/Participation`}>
            ←  go back to Participation 
          </a>
        
        
           <!-- Nav Tabs as Links -->
           <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0 text-center bg-light" href={`/Teacher/${classid}`} role="tab" aria-selected="false">GRADES</a>
              </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link  rounded-0  text-center" href={`/Teacher/${classid}/Prelim`} role="tab" aria-selected="true">PRELIM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0  text-center active disabled" href={`/Teacher/${classid}/Midterm`} role="tab" aria-selected="false">MIDTERM</a>
            </li>
            <li class="nav-item flex-grow-1">
                <a class="nav-link rounded-0  text-center" href={`/Teacher/${classid}/Final`} role="tab" aria-selected="false">FINAL</a>
            </li>
        </ul>

        <ul class="nav nav-tabs rounded-0 d-flex" id="myTab" role="tablist">
          <li class="nav-item flex-grow-1">
              <a class="nav-link  rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Attendance`} role="tab" aria-selected="true">ATTENDANCE</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link active disabled rounded-0 border text-center" href={`/Teacher/${classid}/Midterm/Participation`} role="tab" aria-selected="false">PARTICIPATION</a>
          </li>
          <li class="nav-item flex-grow-1">
              <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Midterm/Quiz`} role="tab" aria-selected="false">QUIZ</a>
          </li>
          <li class="nav-item flex-grow-1">
            <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Midterm/Activity-Project`} role="tab" aria-selected="false">ACTIVITY/PROJECT</a>
        </li>
        <li class="nav-item flex-grow-1">
          <a class="nav-link rounded-0 border  text-center" href={`/Teacher/${classid}/Midterm/Exam`} role="tab" aria-selected="false">EXAM</a>
      </li>
      </ul>

      <div class="d-grid mt-4">
        <button class="btn btn-success" on:click={updateScore}>SAVE</button>
    </div>
      
         <!-- Cards displaying attendance data -->
         <div style="max-height: 42vh; overflow-y: auto; overflow-x: hidden;">
            <table class="table table-bordered table-hover">
            
                <thead class="table-light">
                    
                    <tr>
                        <th scope="col">Name</th>
                        <th scope="col">Score</th>
                        <th scope="col">Perfect Score</th>
                        <th scope="col">Score Type</th>
                    </tr>
                </thead>
                <tbody>
                    {#each participationscoresmidterm as participation}
                    <tr>
                        <td>{participation.Studentlist?.studentinfo?.lastName}{participation.Studentlist?.studentinfo?.firstName}</td>
                        <td>
                            <input
                                type="number"
                                class="form-control"
                                bind:value={participation.score}
                                min="0"
                                on:blur={(e) => {
                                    const enteredScore = Number(e.target.value);
                                    if (enteredScore > participation.perfectscore) {
                                        // Reset the value back to the original score if the entered value is greater than perfect score
                                        e.target.value = participation.score; 
                                        toast.error(`Score cannot be higher than ${participation.perfectscore}`);
                                    } else {
                                        handleInputChange(participation.scoreid, e.target.value);
                                    }
                                }}
                            />
                        </td>
                        <td>{participation.perfectscore}</td>
                        <td>{participation.scoretype}</td>
                    </tr>
                    {/each}
                </tbody>
            </table>
        </div>
        
        

<!-- 
    </div>
      </div> -->

    {/if}



{#if error}
  <div class="alert alert-danger mt-4">
    {error}
  </div>
{/if}

<style>

input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
  }
  
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