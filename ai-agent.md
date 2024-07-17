LLM Inference for AI Agent
We have @tools, we need LLM to call differnt tools to fix different SRE issues. 

If there is a healthcheck fail, like "Ensure ip forwarding is disabled", when detect the healthcheck fail, then call check_nessus_report() to find which server fail for the healthcheck, after find the server, then use tool search_script() to check if there is existing code to fix the health, if yes, then call check_server() tool to verify the server is safe to apply, if yes, call check_conflict() tool to check if there are other tasks running for the server, if yes, quit, if no, then the server is ready to apply, then call fix_healthcheck() tool to apply the fix on target server. 

check_nessus_report() – 查找fail的healthchecks, 输入nessus report, 返回fail的hc
generate_task() – 根据fail的hc生成相应的问题, 输入fail的hc, 输出明确的任务描述.
search_script() – 根据fail hc检索是否有与其匹配的脚本, 输入fail HC, 返回是否有匹配的代码, 
check_server() – 根据server名字进行检索机器详细信息, 输入server name, 返回类型, 服务等, 比如是客户A的non-prod, db2 server. 
check_conflict() – 检查项目最近的event看看是否有冲突的事件, 输入case或event, 返回是或否. 
fix_healthcheck() – 根据匹配的代码执行修复, 输入匹配代码和tag, 目标机器, 
find_evidence() – 根据hc和代码检索相似的已有SR或CR, 返回相似度最高的一个
create_change() – 根据最终solution创建change request,
search_verification() – 根据hc检查匹配的验证脚本, 
verify_healthcheck() – 运行search_verification() 返回的脚本进行验证. 

check_nessus_report() -> generate_task() -> search_script() -> check_server() -> check_conflict() -> fix_healthcheck()
-> find_evidence() -> create_change() -> search_verification() -> verify_healthcheck()