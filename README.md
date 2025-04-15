# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Calendar Versioning](https://calver.org/).

## [23.08.1]
### Changed
- Skip auto-tagging for heatbeat
- Support http://*:8282/gerrit/
- Improve CIofCI for generate release notes
- Fix republish change.json ticket conflict [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/398)
- Remove 3 months older artifacts [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/397)

## [23.07.4]
### Changed
- Fix spec missing release_tags sometimes [CSSRCP-6764](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6764)
- Add release build name filter [FCA_ATFRCPCDC-35509](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-35509)
- Don't post promotion tag with very simple version [FCA_ATFRCPCDC-35497](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-35497)

## [23.07.3]
### Changed
- Add USE_ANCHORE_FOR_DEV_BRANCH=false [CSSRCP-6559](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6559)
- Fix https connect to coop failure [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/393) 
- Rename fci-helmchart to fci-pod for CIofCI
- FCI UT Improvement [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1302)
- Move from fedora-devel to fedora36-devel [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/391) [MR2](https://gitlabe1.ext.net.nokia.com/prime/jenkins-job-shell/-/merge_requests/1171)
- Remove python2 from Jenkins master [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins/-/merge_requests/245)

## [23.07.2]
### Changed
- auto-fill responsible person in correction form in Pronto [FCA_ATFRCPCDC-31218](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-31218)


## [23.07.1]
### Changed
- Add test case for FCI_Coop [FCA_ATFRCPCDC-34670](https://gitlabe1.ext.net.nokia.com/prime/apps/-/merge_requests/105)
- Add approve check for upgrade case [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1300)
- Check pipeline status when rerun PQ [FCA_ATFRCPCDC-34178](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-34178)
- Add back promotion check for pod [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1299)
- Move Coop.py from jenkins-shared-library to jenkins-ci-library for reused by component-ci
- Upgrade fedora36-devel packages for ramp down python2 later

## [23.06.6]
### Changed
- Replace rcppkg with rpmbuild for src.rpm

## [23.06.5]
### Changed
- enable-k8s-dns for CIofCI
- FCI should support correction form selection even when it is container or POD repo[FCA_ATFRCPCDC-34967](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-34967)
- Add package management start time [FCA_ATFRCPCDC-25918](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-25918)
### Fixed
- fix_mergeable [CSSRCP-6569](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6569)
- pipeline triggered via FCI is mismatch with actual job[CSSRCP-6639](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6639)

## [23.06.4]
## Changed
- create restful API interface so Operation team could more easily access E2E delivery metrics[FCA_ATFRCPCDC-24297](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-24297) 

## [23.06.3]
## Changed
- Show blueocean on FCI-Build
- Enable mod_status for FCI Dashboard
- simplify RCP release notes metadata genenrator template[FCA_ATFRCPCDC-32810](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-32810)

## [23.06.2]
### Changed
-  Stop using anonymous user when accessing Artifactory[FCA_ATFRCPCDC-34275](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-34517)

## [23.06.1]
### Changed
- Update correction form with products [FCA_ATFRCPCDC-33769](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-33769) [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1276)
- Change RCP to RCP2.0 for VNF VCU [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1284)
- Show ccs in delivery [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1283)
- Stop anonymous for artifactory except rcp-base [FCA_ATFRCPCDC-34275](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-34275)
### Fixed
- Fix fci_coop not return fail when post error [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/375)

## [23.05.4]
### Added
- Add VIP FCI-Dashboard
### Changed
- FCI branch to show vcu and vdu builds information of containers and pods [FCA_ATFRCPCDC-33147](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-33147)
### Fixed
- Fix pre-check cannot block consolidated gerrit [CSSRCP-6491](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6491)

## [23.05.3]
### Changed
- HelmChart E2E improvement [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1268)
- Visible the mapping of subsystem and pipelines [CSSRCP-6360](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6360)
- Send more coverity checkers(Top25) to Coop [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/369)
- Update prime/apps configuration for artifactory

## [23.05.2]
### Added
- Add gitlab service
- Add artifactory-oss service
### Changed
- Move gitlabe2/prime/ci-library to gitlabe1/prime/jenkins-ci-library
- Move Log_Recheck from puller-jenkins to task-jenkins [CSSRCP-6473](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6473)
- Refactor manual tag rule
### Fixed
- Fix FCI_Coop empty case_DOD.json [CSSRCP-6471](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6471)
- Fix GearmanError: Unable to submit job

## [23.05.1]
### Added
- Update tag user guide when delete manual tag
- fci_release_build_refactor
- Fix "Correction Form" re-selected after edit feature

## [23.04.3]
### Added
- Add FCI_Bump-Pod [FCA_ATFRCPCDC-32891](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-32891) [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1252) [MR2](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/355)

## [23.04.2]
### Added
- Push requirement_id to COOP [FCA_ATFRCPCDC-33066](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-33066) [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/356)
- Push coverity enabled checkers to COOP [FCA_ATFRCPCDC-33214](https://jiradc.ext.net.nokia.com/browse/FCA_ATFRCPCDC-33214) 
### Changed
- Upgrade jenkins [pipeline-graph-view](https://github.com/jenkinsci/pipeline-graph-view-plugin) plugin
- Move FCI documentation from confluence to sphinx [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1260)
- Move FCI_Coop from freestyle to pipeline [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-job-shell/-/merge_requests/1165)
- Improve FCI_Cleaning for obsoleted kojiTags/features/artifacts [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/360)
- Use python3 for featureci/fciagent [MR1](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1262) and [MR2](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/361) 
### Fixed
- Fix get container pipeline error [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/352)
- Fix Merge Request links for gitlab [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1263)
### Removed
- Remove [mypython](https://gitlabe1.ext.net.nokia.com/prime/mypython) libraries

## [23.04.1]
### Added
- create fci_shadow_master for pod [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-job-shell/-/merge_requests/1158)
### Fixed
- Fix FCI_PHASE=container_template [CSSRCP-6335](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6335) [MR](https://gitlabe1.ext.net.nokia.com/prime/featureci/-/merge_requests/1259)
- Fix TagLevel in git notes for container auto-tagging [MR](https://gitlabe1.ext.net.nokia.com/prime/jenkins-shared-library/-/merge_requests/354)


## [23.04.0]
### Added
- Add CHANGELOG.md
- Add gitlabci for pages
### Changed
- Narrow down FCI service nodePort to kubernetes default range 30000-32767 [ports.ini](https://gitlabe1.ext.net.nokia.com/prime/deployment/-/blob/master/credentials/config/ports.ini)
- Support fast deployment by moving jenkins-job-shell to jenkins-shared-library
### Fixed
- Increase 1000 pipelines limitation to 3000 for taas API /api/v1/feature
- Fix forceRcpBaseImage update error
- Fix rpm spec subsystem appears in the code subsystem candidate list [CSSRCP-6324](https://jiradc.ext.net.nokia.com/browse/CSSRCP-6324)

