# LLM Inference for AI Agent

We have @tools, we need LLM to call differnt tools to fix different SRE issues. 

If there is a healthcheck fail, like "Ensure ip forwarding is disabled", when detect the healthcheck fail, then call check_nessus_report() to find which server fail for the healthcheck, after find the server, then use tool search_script() to check if there is existing code to fix the failed healthcheck, if yes, then call check_server() tool to verify the server is safe to apply or not, if yes, call check_conflict() tool to check if there are other tasks running for the server, if yes, quit and plan next time to apply, if no, the server is ready to apply, then call fix_healthcheck_CVE() tool to apply the fix on target server. 

## The tools we provide to LLM are:
* check_nessus_report()
* generate_task()
* search_script()
* check_server()
* check_conflict()
* fix_healthcheck_CVE()
* find_evidence()
* create_change()
* search_verification()
* verify_healthcheck_CVE()

## Details explaination for these tools:

* check_nessus_report() – Check fail healthchecks or CVE, input is nessus report, return fail healthchecks or CVE

* generate_task() – based on failed healthchecks or CVE to return solution to fix it, input is failed healthchecks, return solution tasks.

* search_script() – based on failed healthchecks or CVE to find matched scripts that can fix them, input is failed healthchecks or CVE, return matched automation scripts. 

* check_server() – based on server name to get its details information, input is server name, return the server info, like it's non-prod server, it's db2 server etc.

* check_conflict() – Check if there are conflict activities for the target server, like it's in patching maintenance reboot or not etc., input is server name, return none or event lists

* fix_healthcheck_CVE() – Run matched automation scripts to fix healthchecks or CVE input is matched scrips, tag and target server name

* find_evidence() – based on failed healthchecks or CVE and hostname to search similar services requests or change requests in ticket system, and return the most similarity one. 

* create_change() – create change request, input is failed healthchecks or CVE and hostname and evidence find by find_evidence()

* search_verification() – based on failed healthchecks or CVE to search and match verification scripts 

* verify_healthcheck_CVE() – execute the returned verification scripts by search_verification() to verify. 


Generally we can follow below procedure to handle a failed healthchecks or CVE:

For non-prod servers:

* check_nessus_report() -> generate_task() -> search_script() -> check_server() -> check_conflict() -> fix_healthcheck_CVE() -> search_verification() -> verify_healthcheck_CVE()

For prod servers:

* check_nessus_report() -> generate_task() -> search_script() -> check_server() -> check_conflict() -> find_evidence() -> create_change() -> fix_healthcheck_CVE() -> search_verification() -> verify_healthcheck_CVE()

But based on different status or situation, we need to take different tools and procedure to fix issues. We need LLM to planning and take correct actions based on different scenarios.

## Example:
Detect one healthcheck failed on server db2server1, the failed healthcheck is "Ensure RPC is not enabled", then LLM should think and choose check_nessus_report() tool to get more details info about the failed healthcheck, to get the server name, the solution, if LLM knows the fix scripts exists, then LLM can directly call check_server() tool and check_conflict() tool to decide if it's ready to apply fix. If LLM don't know if the fixing scripts exists or not, it should call search_script() tool to check. If scripts is ready and server is reasdy, and server is non-prod server, no need to create change request, then LLM should directly call fix_healthcheck_CVE() tool to apply on target server. If script and server is ready, but it's prod server, then LLM should call find_evidence() tool and create_change() tool to create change. After applied fixing, LLM should call search_verificatioin() tool and verify_healthcheck_CVE() tool to verify the fixing. 
