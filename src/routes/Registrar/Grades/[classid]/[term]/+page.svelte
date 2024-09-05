<script>
    import { onMount, afterUpdate } from 'svelte';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';

    let classDetails = null;
    let gradelistEntries = [];
    let error = '';
    let term = '';
    let classid = '';

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
        goto(`${selectedTerm}`);
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


        <ul class="nav nav-tabs" id="termTab" role="tablist">
            <li class="nav-item" role="presentation">
                <a href={`${term}/Attendance`} class="nav-link ">Attendance</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${term}/Participation`} class="nav-link ">Participation</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${term}/Quiz`} class="nav-link">Quiz</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${term}/Project`} class="nav-link ">Project/Activity</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${term}/Exam`} class="nav-link">Exam</a>
            </li>
           
        </ul>

        <!-- Display Grades List for the selected term -->
        <!-- <table>
            <thead>
                <tr>
                    <th>Student Name</th>
                  
                    <th>Quiz 1</th>
                    <th>Quiz 2</th>
                    <th>Quiz 3</th>
                    <th>Quiz 4</th>
                </tr>
            </thead>
            <tbody>
                {#each gradelistEntries as entry}
                    <tr>
                        <td>{entry.studentLastName} {entry.studentFirstName}</td>
                   
                        <td>{entry.QuizScore.quiz_1}</td>
                        <td>{entry.QuizScore.quiz_2}</td>
                        <td>{entry.QuizScore.quiz_3}</td>
                        <td>{entry.QuizScore.quiz_4}</td>
                    </tr>
                {/each}
            </tbody>
        </table> -->
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
