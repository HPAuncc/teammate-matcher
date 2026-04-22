# Teammate Matcher — Survey Instrument

Deploy via Google Forms (anonymous, no sign-in required). Export responses as CSV.

---

## Section 1: Context

**Q1.** What class is this team assignment for?
`Short answer`
> ML feature: `course_code` (filter/group variable only, not used in clustering)

**Q2.** What is your year in school?
`Multiple choice`
- Freshman
- Sophomore
- Junior
- Senior
- Graduate

> ML feature: `year` — ordinal encoded (Freshman=1 … Graduate=5)

---

## Section 2: Schedule & Availability

**Q3.** Which days are you generally available to meet with your team? *(Select all that apply)*
`Checkboxes`
- Monday
- Tuesday
- Wednesday
- Thursday
- Friday
- Saturday
- Sunday

> ML features: `avail_mon`, `avail_tue`, `avail_wed`, `avail_thu`, `avail_fri`, `avail_sat`, `avail_sun` — binary (7 columns)

**Q4.** What time(s) of day are you available to meet? *(Select all that apply)*
`Checkboxes`
- Morning (8 AM – 12 PM)
- Afternoon (12 PM – 5 PM)
- Evening (5 PM – 9 PM)
- Late night (9 PM+)

> ML features: `avail_morning`, `avail_afternoon`, `avail_evening`, `avail_latenight` — binary (4 columns)

**Q5.** How many hours per week can you realistically dedicate to group project work?
`Multiple choice`
- Less than 3 hours
- 3–5 hours
- 6–9 hours
- 10+ hours

> ML feature: `weekly_hours` — ordinal encoded (<3=1, 3–5=2, 6–9=3, 10+=4)

**Q6.** Do you prefer to meet in person or remotely?
`Multiple choice`
- In person only
- Remote only
- No preference

> ML feature: `meeting_mode` — one-hot encoded (3 binary columns)

---

## Section 3: Technical Skills

*Rate your confidence in each area from 1 to 5.*
*1 = No experience / 5 = Very confident*

Each question uses a `Linear scale` from 1 to 5.

**Q7.** Python / programming
> ML feature: `skill_python`

**Q8.** Data analysis (pandas, spreadsheets, SQL)
> ML feature: `skill_data_analysis`

**Q9.** Statistics and math
> ML feature: `skill_statistics`

**Q10.** Data visualization (matplotlib, seaborn, Tableau, etc.)
> ML feature: `skill_visualization`

**Q11.** Machine learning / modeling
> ML feature: `skill_ml`

**Q12.** Technical writing and documentation
> ML feature: `skill_writing`

**Q13.** Research and literature review
> ML feature: `skill_research`

**Q14.** Presentations and public speaking
> ML feature: `skill_presenting`

---

## Section 4: Work Style

**Q15.** What role do you naturally gravitate toward in a group?
`Multiple choice`
- I prefer to lead and coordinate
- I prefer to contribute as a specialist
- I'm comfortable in either role
- I prefer to follow clear direction

> ML feature: `role_pref` — ordinal (Follower=1, Contributor=2, Flexible=3, Leader=4)

**Q16.** How do you typically approach deadlines?
`Multiple choice`
- I finish well before the deadline
- I work steadily throughout
- I work best under pressure, closer to the deadline

> ML feature: `deadline_style` — ordinal (Last-minute=1, Steady=2, Early=3)

**Q17.** How do you prefer to communicate with your team? *(Pick your primary preference)*
`Multiple choice`
- Text / iMessage
- Email
- Discord / Slack
- Video call (Zoom, Teams)
- In person

> ML feature: `comm_pref` — one-hot encoded (5 binary columns)

**Q18.** How often do you prefer to check in with teammates?
`Multiple choice`
- Daily
- A few times a week
- Once a week
- Only when necessary

> ML feature: `checkin_freq` — ordinal (As needed=1, Weekly=2, Few/week=3, Daily=4)

**Q19.** When working on a group project, I prefer to...
`Multiple choice`
- Work independently and combine everything at the end
- Collaborate closely together throughout
- A mix — divide tasks but check in regularly

> ML feature: `collab_style` — encoded (Independent=1, Mix=2, Collaborative=3)

**Q20.** My approach to the work tends to be...
`Linear scale` 1–5
- 1 = Big picture / strategy / vision
- 5 = Details / execution / precision

> ML feature: `detail_orientation`

**Q21.** My approach to conflict within a team is usually to...
`Multiple choice`
- Address it directly and immediately
- Talk it out privately first
- Let it resolve naturally over time
- Defer to whoever is leading the group

> ML feature: `conflict_style` — one-hot encoded (4 binary columns)

---

## Section 5: Self-Assessment

**Q22.** What is your GPA range? *(Optional — you may skip this question)*
`Multiple choice`
- Below 2.5
- 2.5 – 3.0
- 3.0 – 3.5
- 3.5 – 4.0
- Prefer not to say

> ML feature: `gpa_band` — ordinal (1–4); "Prefer not to say" treated as missing → imputed with median

**Q23.** What do you most contribute to a team? *(Select up to 2)*
`Checkboxes`
- Technical execution (coding, building)
- Creative ideas and brainstorming
- Organization and planning
- Research and writing
- Keeping team morale up
- Quality checking and editing

> ML features: `contrib_technical`, `contrib_creative`, `contrib_organization`, `contrib_research`, `contrib_morale`, `contrib_qa` — binary (6 columns)

**Q24.** What is your biggest challenge in group projects? *(Select 1)*
`Multiple choice`
- Coordinating schedules
- Unequal workload distribution
- Disagreements on direction or approach
- Communication breakdowns
- Staying motivated

> ML feature: `pain_point` — one-hot encoded (5 binary columns); also used for ethics/bias analysis

---

## Feature Summary

| Section | Questions | # ML Features | Encoding |
|---|---|---|---|
| Context | Q1–Q2 | 1 usable | Ordinal |
| Availability | Q3–Q6 | 14 | Binary + Ordinal |
| Skills | Q7–Q14 | 8 | Continuous (1–5) |
| Work Style | Q15–Q21 | ~15 | Ordinal + One-hot |
| Self-Assessment | Q22–Q24 | ~12 | Ordinal + Binary |
| **Total** | **24 questions** | **~50** | |

---

## Deployment Notes

- Set form to **not collect email addresses**
- Do not require Google sign-in
- Share link via Canvas, class Discord, or email
- Target: 30+ responses minimum for meaningful clustering into teams of 3–4
- Export: Google Forms → Responses tab → Download CSV (.csv)
- Save raw export to `data/raw_survey_responses.csv`
