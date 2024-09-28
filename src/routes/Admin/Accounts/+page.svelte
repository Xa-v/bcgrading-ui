<script>
       import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { jwtDecode } from 'jwt-decode';
    import { fade } from 'svelte/transition';
    import { writable } from 'svelte/store';

    let accounts = [];
    let activeAccounts = [];
    let deletedAccounts = [];
    let error = null;
    let selectedTab = 'active'; // Default tab
    let userRole = '';
    let userID = '';
    const selectedAccount = writable(null);
    let successMessage = ''; // New state for success message
    let bootstrap ='';
    let showDeleteModal = false;
    let accountToDelete = null;
    let accountToDeleteUsername = '';
    let showRestoremodal = false;
    let accountToRestore = null;
    let accountToRestoreUsername = '';

        // Function to handle logout
        function logout() {
      localStorage.removeItem('jwtToken');  // Clear the JWT token
      goto('/Login');  // Redirect to the login page immediately
    }
  

    // Fetch accounts on component mount
    onMount(async () => {

      // clearAllCache();
      const bootstrapModule = await import('bootstrap/dist/js/bootstrap.bundle.min.js');
      bootstrap = bootstrapModule.default;


   
            const token = localStorage.getItem('jwtToken'); // Replace with your actual JWT token

            if (!token) {               
                logout();
                      return;
                  }
  
                  try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;
            userID = decodedToken.id;
            console.log("Role:", userRole);
            console.log("ID:", userID);
            
  
            if (userRole !== 'Admin') {

                goto(`/${userRole}`);
                return;
            }
  
          } catch (error) {
            console.error('Error:', error);
            logout();
        }

            // Fetch active accounts
            const userlist = await fetch('http://localhost:4000/admin', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (userlist.ok) {
                activeAccounts = await userlist.json();
            } else {
                handleUnauthorized('Failed to fetch active accounts');
                return;
            }

            // Fetch inactive (deleted) accounts
            const inactiveUserlist = await fetch('http://localhost:4000/admin/deleted', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (inactiveUserlist.ok) {
                deletedAccounts = await inactiveUserlist.json();
            } else {
                handleUnauthorized('Failed to fetch inactive accounts');
                return;
            }

            // Combine active and inactive accounts for the "All" tab
            accounts = [...activeAccounts, ...deletedAccounts];






    });

  
    async function updateAccount(event) {
        event.preventDefault();
        const formData = new FormData(event.target);
        const accountId = ($selectedAccount && $selectedAccount.id) || null;

        if (!accountId) {
            console.error("Account ID is not available.");
            return;
        }

        try {
            const response = await fetch(`http://localhost:4000/admin/${accountId}`, {
                method: 'PUT',
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(Object.fromEntries(formData))
            });

            if (response.ok) {
                accounts = await fetch('http://localhost:4000/admin', {
                    headers: {
                        'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                    }
                }).then(res => res.json());

                successMessage = 'Update successful!'; // Set the success message
                const modal = document.getElementById('editAccountModal');
                const bsModal = bootstrap.Modal.getInstance(modal);
                setTimeout(() => bsModal.hide(), 1500);
               
            } else {
                error = `Failed to update account: ${response.statusText}`;
            }
        } catch (error) {
            error = `Error updating account: ${error.message}`;
        }
        setTimeout(() => window.location.reload(), 2500); 
    }


    async function changePassword(event) {
        event.preventDefault();
        const formData = new FormData(event.target);
        const accountId = ($selectedAccount && $selectedAccount.id) || null;

        if (!accountId) {
            console.error("Account ID is not available.");
            return;
        }

        try {
            const response = await fetch(`http://localhost:4000/admin/updatepassword/${accountId}`, {
                method: 'PUT',
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(Object.fromEntries(formData))
            });

            if (response.ok) {
                successMessage = 'Password change successful!'; // Set the success message
                const modal = document.getElementById('changePasswordModal');
                const bsModal = bootstrap.Modal.getInstance(modal);
                setTimeout(() => bsModal.hide(), 1500); // Hide modal after a short delay
             


                modal.addEventListener('hidden.bs.modal', () => {
            const form = modal.querySelector('form');
            if (form) {
                form.reset(); // Reset form fields
            }
            // Optionally, clear success and error messages
            successMessage = '';
            error = '';
        });


            } else {
              error = `Failed to change password: ${response.statusText}`;
            }
        } catch (error) {
          error = `Error changing password: ${error.message}`;
        }
       
    }


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
                successMessage = 'Account deleted successfully!';
                setTimeout(() => {
                  bsModal.hide()
                   
                }, 1500);
            } else {
                // Refresh the page in case of failure
                error = `Failed to delete account: Cant delete own account or admin account`;
                setTimeout(() => {
                  bsModal.hide()
                   
                }, 1500);
            }
        } catch (error) {
            error = `Error deleting account: ${error.message}`;
            
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
                successMessage = '';
                error = '';
            }, 3000);
        }
    }
    setTimeout(() => window.location.reload(), 2500); 
  }

  function closeDeleteModal() {
    showDeleteModal = false;
    accountToDelete = null;
    accountToDeleteUsername = '';
  }


  async function openRestoreModal(accountId) {
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
          showRestoremodal = true;
      }
    }
  
    async function confirmRestore() {
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
                  successMessage = 'Account restored successfully!';
                  setTimeout(() => {
                  bsModal.hide()
                   
                }, 1500);
              } else {
                  // Refresh the page in case of failure
                  error = `Failed to delete account: Cant delete own account or admin account`;
                  setTimeout(() => {
                  bsModal.hide()
                   
                }, 1500);
              }
          } catch (error) {
              error = `Error deleting account: ${error.message}`;
              
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
                  successMessage = '';
                  error = '';
              }, 3000);
          }
      }
      setTimeout(() => window.location.reload(), 2500); 
    }
  
    function closeRestoreModal() {
      showRestoremodal = false;
      accountToRestore = null;
      accountToRestoreUsername = '';
    }

</script>
{#if error}
<p>{error}</p>
{/if}

<ul class="nav nav-tabs">
   
    <li class="nav-item">
        <button type="button" class="nav-link {selectedTab === 'active' ? 'active' : ''}" on:click={() => selectedTab = 'active'}>
            Active Accounts
        </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link {selectedTab === 'all' ? 'active' : ''}" on:click={() => selectedTab = 'all'}>
          All Accounts
      </button>
  </li>
    <li class="nav-item">
        <button type="button" class="nav-link {selectedTab === 'deleted' ? 'active' : ''}" on:click={() => selectedTab = 'deleted'}>
            Deleted Accounts
        </button>
    </li>
</ul>

{#if !error}
<div style="max-height: 79vh; overflow-y: auto; overflow-x: hidden;">
  <table class="table table-bordered" >
    
    <!-- Shared thead for 'all' and 'active' tabs -->
    {#if selectedTab === 'all' || selectedTab === 'active'}
    <thead>
      <tr>
        <th>LastName, FirstName</th>
      
        <th>Username</th>
        <th>Role</th>
        <th>Created</th>
        <th>Updated</th>
        <th>Update Info</th>
        <th>Update Password</th>
        <th>Delete</th>
        <th>Status</th>
      </tr>
    </thead>
    {/if}

    <!-- Custom thead for 'deleted' tab -->
    {#if selectedTab === 'deleted'}
    <thead>
      <tr>
        <th>LastName, FirstName</th>
       
        <th>Username</th>
        <th>Role</th>
        <th>Archived Date</th>
        <th>Status</th>
        <th>Restore</th>
      </tr>
    </thead>
    {/if}

    <tbody>
      {#if selectedTab === 'all'}
        {#each accounts as accountinfo}
          <tr>
            <td>{accountinfo.lastName}, {accountinfo.firstName}</td>
            <!-- <td></td> -->
            <td>{accountinfo.username}</td>
            <td>{accountinfo.role}</td>
            <!-- Shortened date format with text-truncate -->
            <td class="text-truncate">{new Date(accountinfo.created).toLocaleString('en-US', { month: 'numeric', day: 'numeric', year: '2-digit', hour: '2-digit', minute: '2-digit' })}</td>
            <td class="text-truncate">{new Date(accountinfo.updated).toLocaleString('en-US', { month: 'numeric', day: 'numeric', year: '2-digit', hour: '2-digit', minute: '2-digit' })}</td>
            <td>
              <button class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#editAccountModal" on:click={() => selectedAccount.set(accountinfo)}>
                Edit
              </button>
            </td>
            <td>
              <button class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#changePasswordModal" on:click={() => selectedAccount.set(accountinfo)}>
                Edit
              </button>
            </td>
            <td>
              <button class="btn btn-danger" on:click={() => openDeleteModal(accountinfo.id)}>Delete</button>
            </td>
            <td>
              {#if accountinfo.isActive}
                <span class="badge text-bg-success d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Active</span>
              {:else}
                <span class="badge text-bg-danger d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Inactive</span>
              {/if}
            </td>
            
            
          </tr>
        {/each}
      {/if}

      {#if selectedTab === 'active'}
        {#each activeAccounts as accountinfo}
          <tr>
            <td>{accountinfo.lastName}, {accountinfo.firstName}</td>
            <!-- <td></td> -->
            <td>{accountinfo.username}</td>
            <td>{accountinfo.role}</td>
            <!-- Shortened date format with text-truncate -->
            <td class="text-truncate">{new Date(accountinfo.created).toLocaleString('en-US', { month: 'numeric', day: 'numeric', year: '2-digit', hour: '2-digit', minute: '2-digit' })}</td>
            <td class="text-truncate">{new Date(accountinfo.updated).toLocaleString('en-US', { month: 'numeric', day: 'numeric', year: '2-digit', hour: '2-digit', minute: '2-digit' })}</td>
            <td>
              <button class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#editAccountModal" on:click={() => selectedAccount.set(accountinfo)}>
                Edit
              </button>
            </td>
            <td>
              <button class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#changePasswordModal" on:click={() => selectedAccount.set(accountinfo)}>
                Edit
              </button>
            </td>
            <td>
              <button class="btn btn-danger" on:click={() => openDeleteModal(accountinfo.id)}>Delete</button>
            </td>
            <td>
              {#if accountinfo.isActive}
                <span class="badge text-bg-success d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Active</span>
              {:else}
                <span class="badge text-bg-danger d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Inactive</span>
              {/if}
            </td>
            

          </tr>
        {/each}
      {/if}

      {#if selectedTab === 'deleted'}
        {#each deletedAccounts as accountinfo}
          <tr>
            <td>{accountinfo.lastName}, {accountinfo.firstName}</td>
           
            <td>{accountinfo.username}</td>
            <td>{accountinfo.role}</td>
            <!-- Shortened date format for archived accounts -->
            <td class="text-truncate">{new Date(accountinfo.dateDeleted).toLocaleString('en-US', { month: 'numeric', day: 'numeric', year: '2-digit', hour: '2-digit', minute: '2-digit' })}</td>
            <td>
              {#if accountinfo.isActive}
                <span class="badge text-bg-success d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Active</span>
              {:else}
                <span class="badge text-bg-danger d-block rounded-0 text-center" style="width: 100px; height: 35px; line-height: 35px; padding: 0;">Inactive</span>
              {/if}
            </td>
            <td>
              <button class="btn btn-success" on:click={() => openRestoreModal(accountinfo.id)}>Restore</button>
            </td>
          </tr>
        {/each}
      {/if}
    </tbody>
  </table>
</div>

{:else}
  <div class="alert alert-danger">{error}</div>
{/if}



<!-- Bootstrap Modal -->
<div class="modal fade" id="editAccountModal" tabindex="-1" aria-labelledby="editAccountModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="editAccountModalLabel">Edit Account</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      {#if $selectedAccount}
      <form id="editAccountForm" on:submit={updateAccount}>
        <div class="modal-body">
          <input type="hidden" name="id" value="{$selectedAccount.id}">
          <div class="row">
            <div class="mb-3 col">
              <label for="firstName" class="form-label">First Name:</label>
              <input type="text" class="form-control" id="firstName" name="firstName" value="{$selectedAccount.firstName}" required>
            </div>
            <div class="mb-3 col">
              <label for="lastName" class="form-label">Last Name:</label>
              <input type="text" class="form-control" id="lastName" name="lastName" value="{$selectedAccount.lastName}" required>
            </div>
          </div>
          <div class="mb-3">
            <label for="username" class="form-label">Username:</label>
            <input type="text" class="form-control" id="username" name="username" value="{$selectedAccount.username}" required>
          </div>
          <div class="mb-3">
            <label for="role" class="form-label">Role:</label>
            <select class="form-select" id="role" name="role">
              <option value="Admin" selected={$selectedAccount.role === 'Admin'}>Admin</option>
              <option value="Student" selected={$selectedAccount.role === 'Student'}>Student</option>
              <option value="Registrar" selected={$selectedAccount.role === 'Registrar'}>Registrar</option>
              <option value="Teacher" selected={$selectedAccount.role === 'Teacher'}>Teacher</option>
            </select>
          </div>

          {#if error}
          <div class="alert alert-danger">{error}</div>
      {/if}
      {#if successMessage}
          <div class="alert alert-success">{successMessage}</div>
      {/if}
        </div>
        <div class="modal-footer">
            <button type="submit" class="btn btn-primary" >Save</button>
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
     
        </div>
      </form>
      {/if}
    </div>
  </div>
</div>


<div class="modal fade" id="changePasswordModal" tabindex="-1" aria-labelledby="changePasswordModalLabel" aria-hidden="true">
  <div class="modal-dialog">
      <div class="modal-content">
          <div class="modal-header">
              <h5 class="modal-title" id="changePasswordModalLabel">Change Password</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <form on:submit={changePassword}>
              <div class="modal-body">
                  <div class="mb-3">
                      <label for="password" class="form-label">New Password</label>
                      <input type="password" class="form-control" id="password" name="password" required>
                  </div>
                  <div class="mb-3">
                      <label for="confirmPassword" class="form-label">Confirm Password</label>
                      <input type="password" class="form-control" id="confirmPassword" name="confirmPassword" required>
                  </div>
                  {#if error}
                      <div class="alert alert-danger">{error}</div>
                  {/if}
                  {#if successMessage}
                      <div class="alert alert-success">{successMessage}</div>
                  {/if}
              </div>
              <div class="modal-footer">
                  <button type="submit" class="btn btn-primary">Save changes</button>
                  <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
               
              </div>
          </form>
      </div>
  </div>
</div>

<!-- Delete Confirmation Modal -->
<div class={`modal fade ${showDeleteModal ? 'show' : ''}`} id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden={!showDeleteModal} style={showDeleteModal ? 'display: block;' : 'display: none;'}>
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="deleteModalLabel">Confirm Deletion</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" on:click={closeDeleteModal}></button>
      </div>
      <div class="modal-body">
        Are you sure you want to delete the account with username:(<strong>{accountToDeleteUsername}</strong>)?
      </div>
      <div class="modal-footer">
        {#if error}
        <div class="alert alert-danger">{error}</div>
    {/if}
    {#if successMessage}
        <div class="alert alert-success">{successMessage}</div>
    {/if}
        <button type="button" class="btn btn-danger" on:click={confirmDelete}>Delete</button>
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" on:click={closeDeleteModal}>Cancel</button>
      </div>
     
    </div>
  </div>
</div>

  <!-- Restore Confirmation Modal -->
  <div class={`modal fade ${showRestoremodal ? 'show' : ''}`} id="restoreModal" tabindex="-1" aria-labelledby="restoreModalLabel" aria-hidden={!showRestoremodal} style={showRestoremodal ? 'display: block;' : 'display: none;'}>
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="restoreModalLabel">Confirm Restore</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" on:click={closeRestoreModal}></button>
        </div>
        <div class="modal-body">
          Are you sure you want to Restore the account with ID <strong>{accountToRestore}</strong> and username <strong>{accountToRestoreUsername}</strong>?
        </div>
        <div class="modal-footer">
          {#if error}
          <div class="alert alert-danger">{error}</div>
      {/if}
      {#if successMessage}
          <div class="alert alert-success">{successMessage}</div>
      {/if}
            <button type="button" class="btn btn-success" on:click={confirmRestore}>Restore</button>
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" on:click={closeRestoreModal}>Cancel</button>
        
        </div>
      </div>
    </div>
  </div>

<style>
   
</style>
