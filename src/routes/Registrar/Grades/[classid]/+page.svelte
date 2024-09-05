<script>
    import { onMount } from 'svelte';
    import { page } from '$app/stores';

    let classDetails = null;
    let gradelistEntries = [];
    let error = '';
   

    // Get the classid and term parameters from the dynamic route
    $: ({ classid, term } = $page.params);

    onMount(async () => {
        try {
            const response = await fetch(`http://localhost:4000/registrar/${classid}`, {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                }
            });

            if (response.ok) {
                const data = await response.json();
                classDetails = data.classDetails;
                gradelistEntries = data.gradelistEntries;
            } else {
                error = `Failed to fetch grades: ${response.statusText}`;
            }
        } catch (err) {
            error = `Error fetching grades: ${err.message}`;
        }
    });

  
</script>

<div >
    {#if error}
        <div class="alert alert-danger">{error}</div>
    {/if}
    
    {#if classDetails}
        <div class="class-details">
            <h2> {classDetails.subjectcode}</h2>
            <h2>{classDetails.semester} Semester {classDetails.year} Teacher ID {classDetails.teacherid}</h2>
        </div>

        <!-- Navigation Links for Each Term -->
        <ul class="nav nav-tabs" id="termTab" role="tablist">
            <li class="nav-item" role="presentation">
                <a href={`${classid}/Prelim`} class="nav-link {term === 'Prelim' ? 'active' : ''}">Prelim</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${classid}/Midterm`} class="nav-link {term === 'Midterm' ? 'active' : ''}">Midterm</a>
            </li>
            <li class="nav-item" role="presentation">
                <a href={`${classid}/Finals`} class="nav-link {term === 'Finals' ? 'active' : ''}">Final</a>
            </li>
        </ul>
        

        <!-- Content based on the selected term -->
     
    {/if}
</div>

<style>
 
    .class-details {
        margin-bottom: 20px;
    }
    .nav-tabs {
        margin-bottom: 20px;
    }
</style>
