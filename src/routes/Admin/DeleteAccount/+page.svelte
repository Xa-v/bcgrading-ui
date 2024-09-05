<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { onMount } from 'svelte';
  import { jwtDecode } from 'jwt-decode';

  let accounts = [];
  let error = '';
  let userRole = '';
  let showDeleteModal = false;
  let accountToDelete = null;
  let accountToDeleteUsername = '';
  let passwordSuccessMessage = '';
  let passwordError = '';

  onMount(async () => {
    if (typeof window !== 'undefined') {
        await import('bootstrap/dist/js/bootstrap.bundle.min.js').then(module => {
            window.bootstrap = module;
        });
    }

    const token = localStorage.getItem('jwtToken');

 

        const decodedToken = jwtDecode(token);
        userRole = decodedToken.role;

        if (userRole !== 'Admin') {
            redirectMessage = `Role '${userRole}' does not have access to this page.`;
            unauthorizedAccess("Redirecting you to your role-specific page.");
            return;
        }

        const userlist = await fetch('http://localhost:4000/admin', {
            headers: {
                'Authorization': `Bearer ${token}`
            }
        });

        if (userlist.ok) {
            accounts = await userlist.json();
        } else {
            error = `Failed to fetch accounts: ${userlist.statusText}`;
        }


        
  });


  async function openDeleteModal(accountId) {
    accountToDelete = accountId;
    try {
        const response = await fetch(`http://localhost:4000/admin/${accountId}`, {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
            }
        });
        if (response.ok) {
            const account = await response.json();
            accountToDeleteUsername = account.username;
        } else {
            error = `Failed to fetch account details: ${response.statusText}`;
        }
    } catch (error) {
        error = `Error fetching account details: ${error.message}`;
    } finally {
        showDeleteModal = true;
    }
  }

  async function confirmDelete() {
    if (accountToDelete) {
        try {
            const response = await fetch(`http://localhost:4000/admin/delete/${accountToDelete}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                }
            });

            if (response.ok) {
                passwordSuccessMessage = 'Account deleted successfully!';
                setTimeout(() => {
                    window.location.reload();
                }, 1500);
            } else {
                // Refresh the page in case of failure
                passwordError = `Failed to delete account: Cant delete own account or admin account`;
                setTimeout(() => {
                    window.location.reload();
                }, 1500);
            }
        } catch (error) {
            passwordError = `Error deleting account: ${error.message}`;
            
        } finally {
            // Hide modal and clear messages
            setTimeout(() => {
                // Directly manipulate DOM to hide the modal
                const modalElement = document.getElementById('deleteModal');
                if (modalElement) {
                    modalElement.classList.remove('show');
                    modalElement.style.display = 'none';
                    // Reset Bootstrap modal's backdrop
                    document.body.classList.remove('modal-open');
                    const backdrop = document.querySelector('.modal-backdrop');
                    if (backdrop) {
                        backdrop.remove();
                    }
                }
                passwordSuccessMessage = '';
                passwordError = '';
            }, 3000);
        }
    }
  }

  function closeDeleteModal() {
    showDeleteModal = false;
    accountToDelete = null;
    accountToDeleteUsername = '';
  }
</script>

<h1 class="text-center">Delete Accounts</h1>

{#if !error}
<table class="table table-bordered mt-5">
  <thead>
    <tr>
      <!-- <th>ID</th> -->
      <th>First Name</th>
      <th>Last Name</th>
      <th>Username</th>
      <th>Role</th>
      <th>Created</th>
      <th>Updated</th>
      <th>Status</th>
      <th>Delete</th>
    </tr>
  </thead>
  <tbody>
    {#each accounts as accountinfo}
      <tr>
        <!-- <td>{accountinfo.id}</td> -->
        <td>{accountinfo.firstName}</td>
        <td>{accountinfo.lastName}</td>
        <td>{accountinfo.username}</td>
        <td>{accountinfo.role}</td>
        <td>{accountinfo.created}</td>
        <td>{accountinfo.updated}</td>
        <td>
          {#if accountinfo.isActive}
            <p class="badge-lg text-center text-bg-success p-2 rounded">Active</p>
          {:else}
            <p class="badge-lg text-center text-bg-danger p-2 rounded">Inactive</p>
          {/if}
        </td>
        <td>
          <button class="btn btn-danger" on:click={() => openDeleteModal(accountinfo.id)}>Delete</button>
        </td>
      </tr>
    {/each}
  </tbody>
</table>
{/if}

{#if error}
  <p>{error}</p>
{/if}


<!-- Delete Confirmation Modal -->
<div class={`modal fade ${showDeleteModal ? 'show' : ''}`} id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden={!showDeleteModal} style={showDeleteModal ? 'display: block;' : 'display: none;'}>
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="deleteModalLabel">Confirm Deletion</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" on:click={closeDeleteModal}></button>
      </div>
      <div class="modal-body">
        Are you sure you want to delete the account username <strong>{accountToDeleteUsername}</strong>?
      </div>
      <div class="modal-footer">
   
        <button type="button" class="btn btn-danger" on:click={confirmDelete}>Delete</button>
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" on:click={closeDeleteModal}>Cancel</button>
      </div>
    </div>
  </div>
</div>

<!-- Success/Error Message -->
{#if passwordSuccessMessage}
  <div class="alert alert-success" role="alert">
    {passwordSuccessMessage}
  </div>
{/if}

{#if passwordError}
  <div class="alert alert-danger" role="alert">
    {passwordError}
  </div>
{/if}

<style>




</style>
