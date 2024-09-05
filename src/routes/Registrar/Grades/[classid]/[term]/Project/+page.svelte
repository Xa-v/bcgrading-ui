<script>
    import { onMount, afterUpdate } from 'svelte';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classDetails = null;
    let gradelistEntries = [];
    let error = '';
    let term = '';
    let classid = '';
    let scoreType = '';

    // Get the classid and term parameters from the dynamic route
    $: ({ classid, term } = $page.params);

    // Fetch class and grade data on mount
    async function fetchData() {
        try {
            const response = await fetch(`http://localhost:4000/registrar/${classid}/${term}`, {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                }
            });

            if (response.ok) {
                const data = await response.json();
                classDetails = data.classDetails;
                gradelistEntries = data.gradelistEntries;
            } else {
                error = `Failed to fetch grades for ${term}: ${response.statusText}`;
            }
        } catch (err) {
            error = `Error fetching grades: ${err.message}`;
        }
    }

    // Run fetchData initially and whenever the term changes
    onMount(fetchData);
    afterUpdate(() => {
        fetchData();
    });

    // Navigate to the selected term while keeping the classid
    function navigateToTerm(selectedTerm) {
        // Replace the route with only the classid and selectedTerm, clearing any score type
        goto(`/Registrar/Grades/${classid}/${selectedTerm}`);
    }

     // Navigate to the selected term and score type
     function navigateToScore( selectedScoreType) {
        goto(`${selectedScoreType}`);
    }
</script>

<div>
    {#if error}
        <div class="alert alert-danger">{error}</div>
    {/if}

    {#if classDetails}
        <div class="class-details">
            <h2>{classDetails.subjectcode}</h2>
            <h2>{classDetails.semester} Semester {classDetails.year} Teacher ID: {classDetails.teacherid}</h2>
            <h3>Term: {term}</h3>
        </div>

        <!-- Navigation Buttons for Each Term -->
        <ul class="nav nav-tabs" id="termTab" role="tablist">
            <li class="nav-item" role="presentation">
                <button
                    class="nav-link {term === 'Prelim' ? 'active' : ''}"
                    type="button"
                    on:click={() => navigateToTerm('Prelim')}
                    role="tab"
                    aria-selected={term === 'Prelim'}
                >
                    Prelim
                </button>
            </li>
            <li class="nav-item" role="presentation">
                <button
                    class="nav-link {term === 'Midterm' ? 'active' : ''}"
                    type="button"
                    on:click={() => navigateToTerm('Midterm')}
                    role="tab"
                    aria-selected={term === 'Midterm'}
                >
                    Midterm
                </button>
            </li>
            <li class="nav-item" role="presentation">
                <button
                    class="nav-link {term === 'Finals' ? 'active' : ''}"
                    type="button"
                    on:click={() => navigateToTerm('Finals')}
                    role="tab"
                    aria-selected={term === 'Finals'}
                >
                    Final
                </button>
            </li>
        </ul>

  <!-- Navigation for Score Types -->
  <ul class="nav nav-tabs" id="scoreTypeTab" role="tablist">
    <li class="nav-item" role="presentation">
        <button
            class="nav-link {scoreType === 'Attendance' ? 'active' : ''}"
            type="button"
            on:click={() => navigateToScore('Attendance')}
            role="tab"
            aria-selected={scoreType === 'Attendance'}
        >
            Attendance
        </button>
    </li>
    <li class="nav-item" role="presentation">
        <button
            class="nav-link {scoreType === 'Participation' ? 'active' : ''}"
            type="button"
            on:click={() => navigateToScore('Participation')}
            role="tab"
            aria-selected={scoreType === 'Participation'}
        >
            Participation
        </button>
    </li>
    <li class="nav-item" role="presentation">
        <button
            class="nav-link {scoreType === 'Quiz' ? 'active' : ''}"
            type="button"
            on:click={() => navigateToScore('Quiz')}
            role="tab"
            aria-selected={scoreType === 'Quiz'}
        >
            Quiz
        </button>
    </li>
    <li class="nav-item" role="presentation">
        <button
            class="nav-link disabled {scoreType === 'Project' ? 'active' : ''}"
            type="button"
            on:click={() => navigateToScore('Project')}
            role="tab"
            aria-selected={scoreType === 'Project'}
        >
            Project/Activity
        </button>
    </li>
    <li class="nav-item" role="presentation">
        <button
            class="nav-link {scoreType === 'Exam' ? 'active' : ''}"
            type="button"
            on:click={() => navigateToScore('Exam')}
            role="tab"
            aria-selected={scoreType === 'Exam'}
        >
            Exam
        </button>
    </li>
</ul>

        <!-- Display Grades List for the selected term -->
        <table>
            <thead>
                <tr>
                    <th></th>
                    <th></th>
                    <th></th>
                    <th>A/P1</th>
                    <th>A/P2</th>
                    <th>A/P3</th>
                    <th>A/P4</th>
                    <th>A/P5</th>
                    <th>A/P6</th>
                    <th>A/P7</th>
                    <th>A/P8</th>
                    <th>A/P9</th>
                    <th>A/P10</th>
                   <th> total</th>
                   <th>%</th>
                </tr>
            </thead>
            <tbody>
                {#each gradelistEntries as yawa}
                    <tr>
                        <td> {yawa.gradeid}</td>
                        <td>{yawa.studentLastName} </td>
                        <td> {yawa.studentFirstName}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_2}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_3}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_4}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_1}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_total}</td>
                        <td>{yawa.ActivityProjectScore.activityproject_percentage}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    {/if}
</div>

<style>
    .class-details {
        margin-bottom: 20px;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
    }
    th {
        background-color: #f2f2f2;
    }
</style>
