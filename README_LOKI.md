# Installing loki

In this doc the `loki` server will be installed on the same machine than the grafana and prometheus services, but this is not mandatory.
`loki` is located in the PackageHub and in the main openSUSE Leap repos.

    zypper in loki

💡 By default Loki only accepts 7-days old log entries.
This could be a issue to analyze older logs for L3 support cases.
This can be modified by increasing the value of `reject_old_samples_max_age` in `/etc/loki/loki.yaml`.

In order to persist Loki's data:

    mkdir /var/lib/loki
    chown loki:loki /var/lib/loki
    sed -i 's/\/tmp\/loki/\/var\/lib\/loki/' /etc/loki/loki.yaml

For now loki has no authentication mechanism.

Start Loki:

    systemctl enable --now loki


# Installing promtail

`promtail` is a daemon scraping the logs and feeding them to `loki`.
We need to install it on the server to get its logs.

On SLES, add the PackageHub repos. On openSUSE there is nothing to add.

Install using `zypper` and give more rights to the loki users to read the logs:

    zypper in promtail
    usermod -a -G root,salt,www,systemd-journal loki

Copy the `promtail.yaml` configuration file to the `/etc/loki/` folder of the server and adjust:

* The client URL to point to your loki server
* The location of the `timestamp` stages to match the timezone of the server

Start the promtail daemon:

    systemctl enable --now promtail

In order to ensure that `promtail` is up an running, point a browser to http://<server>:9081/targets.

# Setup grafana

Open the Grafana, `Data sources` configuration page and add a `Loki` data source with URL `http://localhost:3100`.

As an example upload the dashboard json file in `loki` folder.
It is a copy of the regular Uyuni dashboard with logs added to it.
