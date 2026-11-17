+++
type = "cv"
visibility = "tab"
weight = 2
title = "CV"
date = "2016-01-28"
slug = "resume"

foot = "Be yourself, everyone else is already taken"

[cv]
    profile = "profile_02.jpg"
    [cv.contact]
        enable = true
        [[cv.contact.list]]
            class = "website"
            icon = "fa-globe"
            url = "https://bresilla.com"
            title = "bresilla.com"
        [[cv.contact.list]]
            class = "github"
            icon = "fa-github"
            url = "https://github.com/bresilla"
            title = "github.com/bresilla"
        [[cv.contact.list]]
            class = "linkedin"
            icon = "fa-linkedin"
            url = "https://www.linkedin.com/in/bresilla/"
            title = "linkedin.com/in/bresilla"
        [[cv.contact.list]]
            class = "scholar"
            icon = "fa-globe"
            url = "https://scholar.google.com/citations?user=PYpWgMYAAAAJ"
            title = "Google Scholar"
        [[cv.contact.list]]
            class = "orcid"
            icon = "fa-globe"
            url = "https://orcid.org/0009-0009-5547-2789"
            title = "ORCID 0009-0009-5547-2789"
    [cv.education]
        enable = true
        title = "Education"
        [[cv.education.list]]
            degree = "PhD, Agricultural, Environmental and Food Science and Technology"
            college = "University of Bologna"
            dates = "2015 - 2019"
        [[cv.education.list]]
            degree = "MSc Computer Science"
            college = "Georgia Institute of Technology"
            dates = ""
        [[cv.education.list]]
            degree = "MSc Agricultural Engineering"
            college = "Technical University of Munich"
            dates = ""
        [[cv.education.list]]
            degree = "BSc"
            college = "University of Prishtina"
            dates = ""
    [cv.language]
        enable = true
        title = "Languages"
        [[cv.language.list]]
            language = "Albanian"
            level = "Native"
        [[cv.language.list]]
            language = "English"
            level = "Fluent"
        [[cv.language.list]]
            language = "Italian"
            level = ""
        [[cv.language.list]]
            language = "German"
            level = ""
        [[cv.language.list]]
            language = "Dutch"
            level = "Elementary"
    [cv.interests]
        enable = true
        title = "Interests"
        [[cv.interests.list]]
            interest = "Multi-robot systems"
        [[cv.interests.list]]
            interest = "Programming languages"
        [[cv.interests.list]]
            interest = "Terminal tools and Nix"
        [[cv.interests.list]]
            interest = "Reverse engineering"
        [[cv.interests.list]]
            interest = "Gaming"
    [cv.summary]
        enable = true
        icon = "fa-user"
        title = "SUMMARY"
        summary = """**Robotics researcher at Wageningen University & Research**, working on teams of field robots: peer-to-peer communication, coverage planning and task allocation, so that several machines can share one farm job.

Before Wageningen I did a PhD and a postdoc at the University of Bologna on sensors, robotics and AI for precision orchard management, from real-time fruit detection with CNNs down to the plant physiology underneath (sap flow, water relations, irrigation).

Most of my tools I build myself: ROS 2 and Rust libraries for robot fleets, ISOBUS and machine-communication stacks, simulators, and on the side a programming language and a handful of terminal tools."""
    [cv.experiences]
        enable = true
        icon = "fa-briefcase"
        title = "EXPERIENCE"
        [[cv.jobs.list]]
            position = "Robotics Researcher"
            dates = "2021 - Present"
            company = "Wageningen University & Research, Agro Food Robotics"
            details = """- **Multi-robot coordination** for agriculture: peer-to-peer communication, coverage planning and task allocation for robot fleets (talk at ROSCon 2025, Singapore).
- **Task allocation**: consensus-based bundling for multi-harvester row allocation (*Computers and Electronics in Agriculture*, 2026).
- **Robot and implement communication** over ISOBUS (ISO 11783) for smart weeding in the EU H2020 project Robs4Crops.
- **Field-robot software**, digital twins and simulation with ROS 2, Webots and Isaac Sim.
- **Computer vision**: apple-picking robot evaluation (NXTGEN Hightech), 5D apple pose estimation, AI-based nematode counting."""
        [[cv.jobs.list]]
            position = "Postdoctoral Researcher"
            dates = "2019 - 2021"
            company = "University of Bologna, Department of Agricultural and Food Sciences"
            details = """- **Deep-learning vision for orchards**: fruit detection, crop-load estimation, and CNN-based detection of physiological disorders and maturity in pears.
- **Orchard design for robots**: computer vision in 2D tree-crop training systems."""
        [[cv.jobs.list]]
            position = "PhD Researcher"
            dates = "2015 - 2019"
            company = "University of Bologna"
            details = """Thesis: *Sensors, Robotics and Artificial Intelligence in Precision Orchard Management*.

- **Real-time fruit detection** with single-shot CNNs (*Frontiers in Plant Science*, 2019).
- **Crop-load and chemical-thinning estimation**, and sensor fusion with deep networks for autonomous UAV navigation in orchards.
- **Plant physiology**: sap flow, tree water relations, deficit irrigation and irrigation with treated wastewater."""
    [cv.publications]
        enable = true
        icon = "fa-archive"
        title = "SELECTED PUBLICATIONS"
        intro = "Around thirty papers on agricultural robotics, computer vision and plant physiology. Full list on [Google Scholar](https://scholar.google.com/citations?user=PYpWgMYAAAAJ)."
        [[cv.publications.list]]
            title = "Spatially-aware consensus-based bundling for event-triggered multi-harvester row allocation"
            url = "https://doi.org/10.1016/j.compag.2026.112518"
            tagline = "*Computers and Electronics in Agriculture*, 2026, first author"
        [[cv.publications.list]]
            title = "Enhancing annotations for 5D apple pose estimation through 3D gaussian splatting (3DGS)"
            url = "https://doi.org/10.1016/j.atech.2026.101910"
            tagline = "*Smart Agricultural Technology*, 2026"
        [[cv.publications.list]]
            title = "Implementing a digital twin for flexible operation of agricultural robotics"
            url = "https://doi.org/10.19103/as.2023.0124.28"
            tagline = "*Advances in Agri-Food Robotics*, Burleigh Dodds, 2024"
        [[cv.publications.list]]
            title = "Counting nematodes made easy: leveraging AI-powered automation for enhanced efficiency and precision"
            url = "https://doi.org/10.3389/fpls.2024.1349209"
            tagline = "*Frontiers in Plant Science*, 2024"
        [[cv.publications.list]]
            title = "Smart implements by leveraging ISOBUS: Development and evaluation of field applications"
            url = "https://doi.org/10.1016/j.atech.2023.100341"
            tagline = "*Smart Agricultural Technology*, 2023"
        [[cv.publications.list]]
            title = "Single-Shot Convolution Neural Networks for Real-Time Fruit Detection Within the Tree"
            url = "https://doi.org/10.3389/fpls.2019.00611"
            tagline = "*Frontiers in Plant Science*, 2019, first author"
    [cv.projects]
        enable = true
        icon = "fa-archive"
        title = "OPEN SOURCE"
        intro = "Selected projects; everything else is on [GitHub](https://github.com/bresilla)."
        [[cv.projects.list]]
            title = "robolibs"
            url = "https://github.com/robolibs"
            tagline = "Rust libraries for robot fleets: `peerbus` zero-copy messaging, `syncbot` multi-robot navigation and scheduling, `ondrive` path tracking, `machbus` ISOBUS / J1939 / NMEA 2000."
        [[cv.projects.list]]
            title = "farmbot-ros"
            url = "https://github.com/farmbot-ros"
            tagline = "ROS 2 stack for field robots: coverage planning, task allocation, localization and ISOBUS."
        [[cv.projects.list]]
            title = "gscbba"
            url = "https://github.com/bresilla/gscbba"
            tagline = "Code for the consensus-based multi-harvester row allocation paper."
        [[cv.projects.list]]
            title = "ARES"
            url = "https://github.com/are-standard"
            tagline = "Agriculture Robotics Exchange Standard, with the MACRO crop-operation schema."
        [[cv.projects.list]]
            title = "roc"
            url = "https://github.com/bresilla/roc"
            tagline = "A Rust replacement for the `ros2` command-line tool."
        [[cv.projects.list]]
            title = "fol"
            url = "https://github.com/fol-lang/fol"
            tagline = "A general-purpose systems programming language."
        [[cv.projects.list]]
            title = "magi and balthasar"
            url = "https://github.com/ai-nerv"
            tagline = "A coding agent for Linux and its evidence-backed memory."
        [[cv.projects.list]]
            title = "lule"
            url = "https://github.com/termworks/lule"
            tagline = "ANSI colour scheme generator for the terminal."
        [[cv.projects.list]]
            title = "faqe"
            url = "https://github.com/bresilla/faqe"
            tagline = "The static site generator behind this website."
+++
