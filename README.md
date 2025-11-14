# 🚉 Berlin (BVG) & Brandenburg (VBB) Public Transport Departures for Home Assistant

> ℹ️ [Hier klicken](./docs/liesmich.md) für eine deutsche Beschreibung.

This integration brings **live public transport data** from Berlin and Brandenburg directly into your Home Assistant dashboard. It uses the official VBB API to fetch real-time departures from BVG and VBB stops — including line numbers, destinations, departure times, and delays.

Whether you're commuting, picking up your kids, or just wondering when the next Ringbahn arrives, this integration shows upcoming departures from your selected stops in a clean, readable format.

> 🛠️ This project is a fork of the original Berlin Transport integration by [vas3k](https://github.com/vas3k/home-assistant-berlin-transport) — with enhanced filtering, customization options, and independent maintenance.

![Example of a real-time public transport display at S+U Gesundbrunnen Bhf  station in Berlin, similar to how departures appear in the Home Assistant dashboard.](./docs/screenshots/timetable_card2s.jpg)![Another example](./docs/screenshots/timetable_card3s.jpg)![Another example](./docs/screenshots/timetable_card1s.jpg)

## ✨ Features
- **Real-time departures** from BVG & VBB stops, including line numbers, destinations, delays, and platforms, updated every 90 seconds  
- **Dashboard card integration** for a clean, user-friendly display of upcoming departures
- **Advanced filtering options**: direction, excluded stops, transport types (bus, tram, ferry, etc.)  
- **Customization**: walking time offset, duration window, official VBB line colors, Ringbahn ⟳/⟲ toggle  
- **Localization support** with German and English translations

## 💿 Installation

This integration consists of two components:  
1. **Integration** – fetches real-time departure data from BVG/VBB  
1. **Dashboard card** – displays the data in a clean, user-friendly format  

You need both components. The recommended way to install is via [HACS](https://hacs.xyz/) for easy updates and seamless integration. The setup takes less than 10 minutes.

If you prefer manual installation, please see the [manual installation guide](./docs/manual_install.md).

### 1️⃣ Add repositories to HACS

Open Home Assistant and go to **HACS → Three dots in top right corner → Custom repositories**. Add both of the following repositories:

- `https://github.com/manoth-msft/home-assistant-bvg-vbb-departures` → Type: **Integration**  
- `https://github.comv/manoth-msft/home-assistant-dashboard-card-bvg-vbb-departures` → Type: **Dashboard**

Click **Add**, then reload the HACS page (hit `F5`) to make sure both repositories are available.

### 2️⃣ Search and install components via HACS

1. After refreshing the HACS page, use the search bar and type **bvg**.  
1. Add the following components:
   - **BVG/VBB real-time departures** (Integration)  
   - **Card for BVG/VBB real-time departures integration** (dashboard)
1. Open each entry and select **Download** from the lower‑right corner.
1. Wait for the download to finish. Then refresh the page and restart Home Assistant to activate both components.

### 3️⃣ Add and configure integration

1. Under `Settings → Devices & services`, select **Add Integration**, search for **bvg**, and choose **BVG/VBB Departures**.  
1. Enter the name of the stop you want to monitor. Partial names are supported. Click **OK**, then pick your station from the list of matches and confirm with **OK**.  
1. (Optional) Configure additional parameters such as direction filters, excluded stops, walking time, Ringbahn options, and more.  
   → See [Additional configuration details](#integration
) for a full overview.  
1. Finally, click **OK** and **Done**. The entity will be created and receive its first update within 1–2 minutes.

### 4️⃣ Add card to dashboard
1. Navigate to the dashboard of your choice and add a card.
1. Under **custom cards** you will see the **BVG/VBB departures card**. Click on it.
1. Select the just-created entity and adjust the configuration if necessary. The configuration options are described [here](#card).
1. Save the card and it should update within a few minutes and show you the real-tiome BVG/VBB departures.

Done!

## ⚙️ Additional configuration details
### Integration

Here you can find all optional parameters explained in detail:
- **Direction**: Use `stop_id` to filter departures by direction. Multiple values supported.  
- **Exclude stops**: List of `stop_id` to exclude nearby stops.  
- **Duration**: How many minutes into the future departures are fetched (default: 10).  
- **Walking time**: Offset to hide unreachable departures.  
- **Enable official VBB line colors**: Use official colors instead of predefined ones.  
- **Hide Ringbahn ⟳/⟲**: Optionally hide clockwise/counter‑clockwise Ringbahn services.  
- **Remove (Berlin) suffix**: Automatically strip the suffix from station names.  
- **Transport options**: Choose which transport types (bus, ferry, etc.) to show or hide.  

### Card

- show_cancelled: true # show or hide the cancelled departures. When not defined or true, the  cancelled departures will be shown as struk-through.
- show_delay: true # show or hide the delay if reported. When not defined or true, the delay will be shown next to the departure time.
- show_absolute_time: true # show the absolute time till departure.
- show_relative_time: true # show the relative time till departure.
- include_walking_time: true # subtract walking time to the stop from the relative time to the departure.

## ❓ FAQ
### How do I find a `stop_id`?

Unfortunately, I didn't have time to figure out a proper user-friendly approach of adding new components to Home Assistant, so you will have to do some routine work of finding the IDs of the nearest transport stops to you. Sorry about that :)

Simply use this URL: **https://v6.vbb.transport.rest/locations?results=1&query=alexanderplatz**

Replace `alexanderplatz` with the name of your own stop.

![](./docs/screenshots/stop-id-api.jpg)

> 🧐 **Pro tip:**
> You can also use their [location-based API](https://v6.vbb.transport.rest/api.html#get-stopsnearby) to find all stops nearby using your GPS coordinates.


## 👩‍💻 Technical details

This sensor uses VBB Public API to fetch all transport information.

- API docs: [https://v6.vbb.transport.rest/api.html](https://v6.vbb.transport.rest/api.html)
- Rate limit: 100 req/min
- Format: [HAFAS](https://github.com/public-transport/hafas-client)

The component updates every 90 seconds, but it makes a separate request for each stop. That's usually enough, but I wouldn't recommend adding dozens of different stops so you don't hit the rate limit.

The VBB API occasionally returns 503 or timeout errors due to temporary instability. While these responses are not uncommon, they do not affect the functionality of the integration beyond generating warning messages in the Home Assistant logs. At present, there is no reliable workaround for this behavior.

After fetching the API, it creates one entity for each stop and writes 10 upcoming departures into `attributes.departures`. The entity state is not really used anywhere, it just shows the next departure in a human-readable format. If you have any ideas how to use it better — welcome to Github Issues.

 If you want to change options later on, just run through the steps again with the same stop. The previous entity will be overwritten automatically.

> 🤔
> In principle, the HAFAS format is standardized in many other cities too, so you should have no problem adapting this component to more places if you wish. Check out [transport.rest](https://transport.rest/) for an inspiration.

## ❤️ Contributions

Contributions are welcome. Feel free to [open a PR](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/pulls) and send it to review. If you are unsure, [open an Issue](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/issues) and ask for advice.

## 🐛 Bug reports and feature requests

Since this is my small hobby project, I cannot guarantee you a 100% support or any help with configuring your dashboards. I hope for your understanding.

- **If you find a bug** - open [an Issue](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/issues) and describe the exact steps to reproduce it. Attach screenshots, copy all logs and other details to help me find the problem.
- **If you're missing a certain feature**, describe it in Issues or try to code it yourself.

## 👮‍♀️ License

- [MIT](./LICENSE.md)
