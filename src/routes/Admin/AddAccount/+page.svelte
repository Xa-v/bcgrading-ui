<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let firstName = '';
    let lastName = '';
    let username = '';
    let password = '';
    let confirmPassword = '';
    let role = ''; // Default role
    let message = '';
    let successMessage = ''; // For success message
    let errorMessage = ''; // For error message

    const roles = ['Admin', 'Registrar', 'Student', 'Teacher'];

    async function handleSubmit() {
        if (password !== confirmPassword) {
            message = "Passwords do not match.";
            return;
        }

        // Retrieve JWT token from localStorage
        const token = localStorage.getItem('jwtToken');

        try {
            const response = await fetch('http://localhost:4000/admin', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}` // Include JWT token
                },
                body: JSON.stringify({
                    firstName,
                    lastName,
                    username,
                    password,
                    confirmPassword,
                    role
                })
            });
            // const result = await response.json();
            successMessage = 'Account created successfully!';
            errorMessage = ''; // Clear any previous error messages

            // Optionally, clear the form fields
            firstName = '';
            lastName = '';
            username = '';
            password = '';
            confirmPassword = '';
            role = '';

            // Redirect after a short delay
            setTimeout(() => {
                window.location.reload();
            }, 3000); // 3 seconds delay
        } catch (error) {
            errorMessage = error.message || 'Error creating account';
            successMessage = ''; // Clear any previous success messages
        }
    }

   
</script>


<h4 class="text-center">Add Account</h4>

<div class="row justify-content-center align-items-center h-100">
    <div class="col-4">
        <div class="card shadow-2-strong card-registration" style="border-radius: 15px;">
            <div class="card-body p-4">
                <!-- Success and Error Messages -->
                {#if errorMessage}
                    <div class="alert alert-danger">{errorMessage}</div>
                {/if}
                {#if successMessage}
                    <div class="alert alert-success">{successMessage}</div>
                {/if}

<form class="form-inline" on:submit|preventDefault={handleSubmit}>
    <div class="row">
        <div class="col-md-6 mb-3"> 
            <div data-mdb-input-init class="form-outline">
                <label class="form-label" for="firstName" >First Name:</label>
                <input type="text" class="form-control form-control-sm" placeholder="First Name" style="position:relative;"  bind:value={firstName} required />
            </div></div>    
                 
    <div class="col-md-6 mb-3">
        <div data-mdb-input-init class="form-outline">
            <label class="form-label" for="lastName">Last Name:</label>
        <input id="lastName" type="text" class="form-control form-control-sm" placeholder="Last Name" bind:value={lastName} required />
        </div></div>
    
    </div>  
   
    <div class="row">
        <div class="col-md-12 mb-3">
            <div data-mdb-input-init class="form-outline">
        <label class="form-label" for="username">Username:</label>
        <input class="form-control form-control-sm" placeholder="username" id="username" type="text"  bind:value={username} required />
    </div>
    </div>
    </div>



<div class="row">
    <div class="col-md-6 mb-3">
        <div data-mdb-input-init class="form-outline">
        <label class="form-label" for="password">Password:</label>
        <input class="form-control form-control-sm" placeholder="password" id="password" type="password" bind:value={password} required />
    </div></div>


<div class="col-md-6 mb-3">
    <div data-mdb-input-init class="form-outline">
    <label class="form-label" for="confirmPassword">Confirm Password:</label>
    <input class="form-control form-control-sm" placeholder="ConfirmPassword" id="confirmPassword" type="password" bind:value={confirmPassword} required />
    </div></div>


</div>
<div class="row">
    <div class="col-10 ">
        <label class="form-label select-label" for="role">Role:</label>
        <select class="form-select" id="role"  bind:value={role}>
            {#each roles as r}
                <option value={r}>{r}</option>
            {/each}
        </select>
    </div>
</div>


    <div class="mt-4 pt-2">
        <button data-mdb-ripple-init class="btn btn-primary btn-sm" type="submit">Create Account</button>
      </div>

</form>
















         
</div>

</div>

</div>

</div>



<style>
 
 input:focus::placeholder {
  color: transparent;
}

</style>