# eval_stig

An Ansible role that stages the Evaluate-STIG package and answer files on Red Hat family Linux or Windows hosts, then configures a monthly assessment.

## Behavior

- On Red Hat family systems, extracts the package under `/opt/eval_stig` by default, copies answer files into its `AnswerFiles` directory, makes `Evaluate-STIG_Bash.sh` executable, and creates a root cron job.
- On Windows, extracts the package under `C:\tmp` by default, copies answer files into its `AnswerFiles` directory, and registers a monthly scheduled task running `Evaluate-STIG.ps1` as `SYSTEM`.
- Other operating system families are skipped.
- The assessment is invoked with `--Output CKLB` on both platforms.

The package archive and answer files are read from the Ansible controller. Ensure both source paths exist there and that the controller can read them. The default answer file path is a directory, so its contents are copied to the destination.

## Requirements

- Ansible with the `ansible.windows` and `community.windows` collections installed (see `requirements.yml`).
- SSH and Python prerequisites for managed Linux hosts, and a supported Windows connection such as WinRM for Windows hosts.
- The Evaluate-STIG archive and answer files available on the controller.
- Privilege escalation on Linux to create files under `/opt` and install a root cron job. The Windows task is configured to run as `SYSTEM`.

Install the collections with:

```sh
ansible-galaxy collection install -r requirements.yml
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `eval_stig_version` | `1.2607.2` | Evaluate-STIG release version; used to construct the archive and extracted directory names. |
| `eval_stig_latest` | `/files/public/software/eval-stig/evaluate-stig-{{ eval_stig_version }}.tar.gz` | Controller-side path to the release archive. |
| `answer_files_src` | `/files/public/software/eval-stig/answer_files/` | Controller-side path to the answer files. |
| `linux_eval_stig_dest` | `/opt/eval_stig` | Destination root for the package on Linux. |
| `win_eval_stig_dest` | `C:\tmp` | Destination root for the package on Windows. |
| `linux_answer_files_dest` | `/opt/eval_stig/evaluate-stig-{{ eval_stig_version }}/Src/Evaluate-STIG/AnswerFiles` | Answer file destination on Linux. |
| `win_answer_files_dest` | `C:\tmp\evaluate-stig-{{ eval_stig_version }}\Src\Evaluate-STIG\AnswerFiles` | Answer file destination on Windows. |
| `eval_stig_start_boundary` | `2026-10-01T00:00:00` | Windows scheduled task start boundary in ISO 8601 format (`YYYY-MM-DDTHH:MM:SS`). |
| `linux_eval_stig_cron_schedule` | `monthly` | Linux cron special time: `monthly`, `weekly`, `daily`, `hourly`, or `reboot`. |

Override defaults in inventory or play variables when your artifact repository or desired installation paths differ. Keep the destination paths aligned with the extracted package version.

## Example

```yaml
- name: Configure monthly Evaluate-STIG assessments
  hosts: eval_stig_targets
  become: true
  roles:
    - role: eval_stig
      vars:
        eval_stig_version: "1.2607.2"
        eval_stig_latest: "/srv/software/evaluate-stig-1.2607.2.tar.gz"
        answer_files_src: "/srv/software/eval-stig/answer_files/"
        eval_stig_start_boundary: "2026-10-01T00:00:00"
        linux_eval_stig_cron_schedule: "monthly"
```

`become: true` is needed for Linux hosts. Windows hosts use the Windows connection configured in inventory; the role does not set connection credentials.

## Scheduling notes

The Linux cron job uses `linux_eval_stig_cron_schedule`, whose default is `monthly`. The Windows task runs on the last day of each month and uses `eval_stig_start_boundary` as its start boundary. Set that value to the desired ISO 8601 date and time for your deployment. The role also does not remove older package versions when `eval_stig_version` changes.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for the full license text.
