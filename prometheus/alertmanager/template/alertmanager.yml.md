```yml
global:
  resolve_timeout: 5m

templates:
  - '/etc/alertmanager/template/*.tmpl'  # 模板的地址

route:
  receiver: 'default-receiver'
  group_by: ['alertname', 'dc']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 4h
  routes:

  # --------------------
  # 云服务质量报警群:QosAlert
  # CSQA_QosAlert
  # ====================
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 2h
    continue: true
    match_re:
      alertname: cpu_load_over_threshold
      instance: "ln.*"
  - receiver: 'CSQA_QosAlert'
    group_by: ['alertname', 'dc', 'jobid', 'user']
    repeat_interval: 1d
    continue: true
    match_re:
      alertname: job_run_with_ethernet|orphan_job
  - receiver: 'CSQA_QosAlert'
    group_by: ['alertname', 'dc', 'user']
    repeat_interval: 1d
    continue: false
    match_re:
      alertname: al_job_which_request_more_than_gres_nnode
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname','user','path']
    group_wait: 2m
    group_interval: 2m
    repeat_interval: 2h
    match:
      alertname: lustre_used_more_than_quota
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname','user','jobid']
    group_wait: 2m
    group_interval: 2m
    repeat_interval: 2h
    match:
      alertname: qos_ss_job_with_node
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname','user']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 2h
    match_re:
      alertname: jobs_empty_running_or_not_calculated|jobs_running_flops_plummet
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname','target']
    group_wait: 2m
    group_interval: 1m
    repeat_interval: 2h
    match:
      alertname: dc_target_util_high
  - receiver: 'CSQA_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 2m
    group_interval: 1m
    repeat_interval: 2h
    match:
      alertname: dc_snode_iowait_high
  - receiver: 'CSQA_QosAlert'
    group_by: ['alertname', 'dc', 'jobid']
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: al_failed_jobs_alert
    
  # --------------------
  # 超算云服务质量监测组:质量监测机器人
  # SCSQMG_QualityMonitorRobot
  # ====================
  # - receiver: 'SCSQMG_QualityMonitorRobot'
  #   group_by: ['alertname', 'dc', 'jobid', 'user']
  #   repeat_interval: 1d
  #   continue: true
  #   match_re:
  #     alertname: al_high_lustre_iops_of_job|al_high_lustre_iops_of_user
  - receiver: 'SCSQMG_QualityMonitorRobot'
    group_by: ['alertname','dc','instance']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 1h
    continue: false
    match_re:
      alertname: Storage_host_ethernet_down
  - receiver: 'SCSQMG_QualityMonitorRobot'
    group_by: ['alertname','dc','instance','jobid']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 4h
    continue: false
    match_re:
      alertname: ss_job_memory_used_alert
  - receiver: 'SCSQMG_QualityMonitorRobot'
    group_by: ['dc','alertname']
    group_wait: 30s
    group_interval: 1m
    repeat_interval: 2h
    continue: true
    match_re:
      alertname: disk_iops_over_threshold_bscca|disk_iops_over_threshold_bsccd|disk_iops_over_threshold_cstc9
  - receiver: 'SCSQMG_QualityMonitorRobot'
    group_by: ['alertname', 'dc', 'queue']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 6h
    continue: false
    match_re:
      alertname: queue_clurm_emergency_not_exsit|node_state_in_queue_clurm_emergency_is_abnormal

  # --------------------
  # 电厂运维群：QosAlert
  # DCOperation_QosAlert
  # ====================
  # - receiver: 'DCOperation_QosAlert'
  #   group_by: ['alertname', 'dc', 'jobid', 'user']
  #   repeat_interval: 1d
  #   continue: true
  #   match_re:
  #     alertname: al_high_lustre_iops_of_job|al_high_lustre_iops_of_user
  
  # --------------------
  # qos 测试群:sion
  # qos_test_sion
  # ====================
  - receiver: 'qos_test_sion'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: public_over_threshold|load_over_threshold|mem_load_over_threshold|node_temp_over_threshold
  - receiver: 'qos_test_sion'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 2h
    continue: false
    match_re:
      alertname: ib_transRate_error_cstc9|ib_transRate_error_bscc_a_d|ib_receivedLosePacketsRate_error|ib_transmittedLosePacketsRate_error|ib_down_error

  - receiver: 'qos_test_sion'
    group_by: ['dc','alertname']
    group_wait: 0s
    group_interval: 10s
    repeat_interval: 2m
    match:
      alertname: qos_docs_job


  # - receiver: 'qos_test_sion'
  #   group_by: ['alertname', 'dc', 'jobid']
  #   repeat_interval: 5m
  #   group_wait: 5s
  #   match_re:
  #     alertname: al_high_lustre_iops_of_job_man_test
  # - receiver: 'qos_test_sion'
  #   group_by: ['alertname', 'dc', 'jobid']
  #   repeat_interval: 4h
  #   group_wait: 10s
  #   continue: true
  #   match_re:
  #     alertname: al_failed_jobs_alert
  #     state: FAILED
  
  # --------------------
  # (严重)QoS报警群: QosAlert
  # QosInternal_QosAlert
  # ====================
  - receiver: 'QosInternal_QosAlert'
    group_by: ['alertname', 'dc', 'name', 'node', 'timepoint']
    repeat_interval: 1h
    group_wait: 10s
    group_interval: 10s
    match_re:
      alertname: al_err_logs_discovered
  - receiver: 'QosInternal_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 1h
    continue: false
    match_re:
      alertname: al_high_lustre_disk_util
  - receiver: 'QosInternal_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 30m
    continue: true
    match:
      alertname: instance_down
  - receiver: 'QosInternal_QosAlert'
    group_by: ['alertname', 'dc']
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: al_storage_dg_total_disk_not_match|al_storage_dg_size_not_match|al_storage_dg_raw_size_not_match|al_storage_dg_disk_not_healthy|al_storage_da_total_disk_not_match|al_storage_da_dg_count_not_match|al_storage_da_backup_disk_not_match|al_storage_server_down|al_storage_server_cp_down

  # --------------------
  # (非常严重)QoS报警群: QosAlert
  # SC_QosInternal_QosAlert
  # ====================
  - receiver: 'SC_QosInternal_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 1h
    continue: false
    match_re:
      alertname: InstanceDown

  # --------------------
  # (严重)IE报警群:QosAlert
  # IE_QosAlert
  # ====================

  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 0s
    group_interval: 1m
    repeat_interval: 2h
    match:
      alertname: dc_storage_util_change_too_much

  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 10s
    group_interval: 1m
    repeat_interval: 2h
    match:
      alertname: illegal_use_compute_node

  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 0s
    group_interval: 1m
    repeat_interval: 2h
    match:
      alertname: dc_snode_lnet_state_abnormal
  - receiver: 'IE_QosAlert'
    group_by: ['alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 2h
    continue: true
    match_re:
      alertname: al:dc_usage_change_too_much
  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: cpu_load_over_threshold
  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname', 'instance']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: HT_enable_on_compute_node
  - receiver: 'IE_QosAlert'
    group_by: ['alertname', 'dc', 'user']
    repeat_interval: 4h
    continue: false
    match_re:
      alertname: al_temperature
  - receiver: 'IE_QosAlert'
    group_by: ['alertname', 'dc']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 2h
    continue: false
    match_re:
      alertname: dedicated_VM_Stat_err
  - receiver: 'IE_QosAlert'
    group_by: ['dc','alertname']
    group_wait: 10s
    continue: false
    group_interval: 1m
    repeat_interval: 2h
    match_re:
      alertname: al_bubble_job|dc_time_deviation_too_large
  - receiver: 'IE_QosAlert'
    group_by: ['alertname', 'dc', 'srorage']
    group_wait: 10s
    continue: false
    group_interval: 1m
    repeat_interval: 2h
    match_re:
      alertname: al:lustre_iops_pressure_dispersion_rate_too_low

  # --------------------
  # (非常严重)IE报警群:IE服务报警机器人
  # SC_IE_QosAlert
  # ====================
  - receiver: 'SC_IE_QosAlert'
    group_by: ['dc','alertname','instance']
    group_wait: 0m
    group_interval: 1m
    repeat_interval: 24h
    match_re:
      alertname: login_nodeRestart|storage_nodeRestart
  - receiver: 'SC_IE_QosAlert'
    group_by: ['alertname','dc']
    group_wait: 10s
    group_interval: 1m
    repeat_interval: 1h
    continue: true
    match:
      alertname: storageEventsAlert

  # --------------------
  # 超算云研发管理周例会:云服务QoS报警机器人
  # SCDMWM_CloudServiceQosAlert
  # ====================
  - receiver: 'SCDMWM_CloudServiceQosAlert'
    group_by: ['dc','alertname','instance']
    group_wait: 1m
    group_interval: 1m
    repeat_interval: 24h
    match:
      alertname: login_nodeRestart
  
  # --------------------
  # （保密）VIP包节点用户资源保障: QosAlert
  # VIPUserResProtection_QosAlert
  # ====================
  - receiver: 'VIPUserResProtection_QosAlert'
    group_by: ['alertname', 'dc', 'node']
    repeat_interval: 24h
    continue: false
    match_re:
      alertname: al_node_in_vip_queue_is_unhealthy

  # --------------------
  # IOPS专属报警群: QosAlert
  # IOPSExclusiveAlert_QosAlert
  # ====================
  - receiver: 'IOPSExclusiveAlert_QosAlert'
    group_by: ['alertname', 'dc', 'jobid', 'user']
    repeat_interval: 1d
    continue: true
    match_re:
      alertname: al_high_lustre_iops_of_job|al_high_lustre_iops_of_user
  - receiver: 'IOPSExclusiveAlert_QosAlert'
    group_by: ['alertname', 'dc', 'jobid']
    repeat_interval: 5m
    group_wait: 5s
    match_re:
      alertname: al_high_lustre_iops_of_job_man

  # --------------------
  # 电厂巡检群: QosAlert
  # DCInspection_QosAlert
  # ====================
  - receiver: 'DCInspection_QosAlert'
    group_by: ['alertname', 'dc', 'jobid']
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: "al_failed_jobs_alert"
      state    : "NODE_FAIL"              # 对电厂运维群，做相关筛选

  # --------------------
  # qos值班交流群: QosAlert
  # DCInspection_QosAlert
  # ====================
  - receiver: 'QosOnDutyComm_QosAlert'
    group_by: ['alertname', 'dc', 'jobid']
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: "al_failed_jobs_alert"
      state    : "NODE_FAIL"
  - receiver: 'QosOnDutyComm_QosAlert'
    group_by: ['alertname', 'dc']
    repeat_interval: 4h
    continue: true
    match_re:
      alertname: al_storage_dg_total_disk_not_match|al_storage_dg_size_not_match|al_storage_dg_raw_size_not_match|al_storage_dg_disk_not_healthy|al_storage_da_total_disk_not_match|al_storage_da_dg_count_not_match|al_storage_da_backup_disk_not_match|al_storage_server_down|al_storage_server_cp_down
  
  # --------------------
  # 孤儿作业报警时效: QosAlert
  # OrphanJobAlert_QosAlert
  # ====================
  - receiver: 'OrphanJobAlert_QosAlert'
    group_by: ['alertname', 'dc', 'jobid', 'user']
    repeat_interval: 1d
    continue: False
    match_re:
      alertname: orphan_job

  # --------------------
  # 可控电厂异常作业报警群（IB网络相关）: QosAlert
  # DcAbnormalJob_IBNet_QosAlert
  # ====================
  - receiver: 'DcAbnormalJob_IBNet_QosAlert'
    group_by: ['alertname', 'dc', 'node']
    repeat_interval: 5m
    match_re:
      alertname: al_ib_err_counter_increase_on_running_node

receivers:
# 云服务质量报警群:QosAlert
- name: 'CSQA_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=f5597100-72f7-4bee-a973-6a0313560fec'
# (严重)IE报警群:QosAlert
- name: 'IE_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=c9ea4ad9-d4c5-4c73-9cdc-e4a087e2da80'
# (非常严重)IE报警群:IE服务报警机器人
- name: 'SC_IE_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=e9774126-e0ad-4901-a96b-3bde79489350'
# qos 测试群:sion
- name: 'qos_test_sion'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=b8088b27-2d41-4d12-be30-488d9236853d'
# 超算云服务质量监测组:质量监测机器人
- name: 'SCSQMG_QualityMonitorRobot'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=a1c513a0-ff37-4846-a5f7-78f093f24d12'
# 电厂运维群：QosAlert
- name: 'DCOperation_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=90885a07-0127-46f9-a0ac-453a0e1e3de8'
# (严重)QoS报警群: QosAlert
- name: 'QosInternal_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=8352fbcc-c9f9-48ee-998e-169436aa8858'
# (非常严重)QoS报警群:QOS服务报警机器人
- name: 'SC_QosInternal_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=9d88ec80-cf86-4949-8fff-a6fc8e2059d3'
# 超算云研发管理周例会:云服务QoS报警机器人
- name: 'SCDMWM_CloudServiceQosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=82ca348e-d027-48fa-aed5-2a830e2f58c4'
# （保密）VIP包节点用户资源保障: QosAlert
- name: 'VIPUserResProtection_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=98c0407b-985e-4e07-9fd3-b3ec1944bc20'
# IOPS专属报警群: QosAlert
- name: 'IOPSExclusiveAlert_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=dc78926b-7aa6-4129-8f0e-67ba04b7c2e0'
# 电厂巡检群：QosAlert
- name: 'DCInspection_QosAlert'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/am/wx/robot?key=bcfb3c8d-9608-41c1-bef9-b0fc405388f4'
# qos值班交流群: QosAlert
- name: 'QosOnDutyComm_QosAlert'
  webhook_configs:
  - url: "http://127.0.0.1:5000/am/wx/robot?key=8cc260e8-ad9d-4afb-b81a-d217f3411f84"
# 孤儿作业报警时效: QosAlert
- name: 'OrphanJobAlert_QosAlert'
  webhook_configs:
  - url: "http://127.0.0.1:5000/am/wx/robot?key=39242a81-4baf-4baf-b32a-33cad65d675b"
# 可控电厂异常作业报警群（IB网络相关）: QosAlert
- name: 'DcAbnormalJob_IBNet_QosAlert'
  webhook_configs:
  - url: "http://127.0.0.1:5000/am/wx/robot?key=0375a520-4b60-4b5e-81fe-b3e84974b89b"

# 默认接收人，用以忽略所有消息
- name: 'default-receiver'
  webhook_configs:
  - url: 'http://127.0.0.1:5000/blackhole'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

