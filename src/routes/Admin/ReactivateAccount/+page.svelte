<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { jwtDecode } from 'jwt-decode';
  
    let accounts = [];
    let error = '';
    let userRole = '';
    let showDeleteModal = false;
    let accountToRestore = null;
    let accountToRestoreUsername = '';
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

  
          const userlist = await fetch('http://localhost:4000/admin/deleted', {
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
      accountToRestore = accountId;
      try {
          const response = await fetch(`http://localhost:4000/admin/${accountId}`, {
              headers: {
                  'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
              }
          });
          if (response.ok) {
              const account = await response.json();
              accountToRestoreUsername = account.username;
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
      if (accountToRestore) {
          try {
              const response = await fetch(`http://localhost:4000/admin/reactivate/${accountToRestore}`, {
                  method: 'PUT',
                  headers: {
                      'Content-Type': 'application/json',
                      'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                  }
              });
  
              if (response.ok) {
                  passwordSuccessMessage = 'Account restored successfully!';
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
              }, 1000);
          }
      }
    }
  
    function closeDeleteModal() {
      showDeleteModal = false;
      accountToRestore = null;
      accountToRestoreUsername = '';
    }
  </script>
  
  <h1 class="text-center"> Restore Accounts</h1>
  
  {#if !error}
  <table class="table table-bordered mt-5">
    <thead>
      <tr>
        <th>ID</th>
        <th>First Name</th>
        <th>Last Name</th>
        <th>Username</th>
        <th>Role</th>
        <th>Created</th>
        <th>Updated</th>
        <th>Status</th>
        <th>I balik</th>
      </tr>
    </thead>
    <tbody>
      {#each accounts as accountinfo}
        <tr>
          <td>{accountinfo.id}</td>
          <td>{accountinfo.firstName}</td>
          <td>{accountinfo.lastName}</td>
          <td>{accountinfo.username}</td>
          <td>{accountinfo.role}</td>
          <td>{accountinfo.created}</td>
          <td>{accountinfo.updated}</td>
          <td>
            {#if accountinfo.isActive}
              <p class="badge-lg text-center text-bg-success">Active</p>
            {:else}
              <p class="badge-lg text-center text-bg-danger">Inactive</p>
            {/if}
          </td>
          <td>
            <button class="btn btn-success" on:click={() => openDeleteModal(accountinfo.id)}>Restore</button>
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
          <h5 class="modal-title" id="deleteModalLabel">Confirm Restore</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" on:click={closeDeleteModal}></button>
        </div>
        <div class="modal-body">
          Are you sure you want to Restore the account with ID <strong>{accountToRestore}</strong> and username <strong>{accountToRestoreUsername}</strong>?
        </div>
        <div class="modal-footer">
            <button type="button" class="btn btn-success" on:click={confirmDelete}>Restore</button>
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
  