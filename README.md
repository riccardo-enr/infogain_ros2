# Information Theoretic Mapping ROS 2 Library

This library containes ROS 2 nodes and utilities for
information-theoretic mapping. This is used as a submodule for an I-MPPI
controller.

For now the goal is to have algorithms such as:

- CSQMI - Cauchy-Schwarz Quadratic Mutual Information
- FSMI - Fast Shannon Mutual Information
- FCMI - Fast Continous Mutual Information

> [!WARNING]
> This is used as a submodule for another meta repository. Which has been left
> private for now.

All the nodes (algorithms) should be GPU accelerated. Since they are used in pairing with an I-MPPI controller that should be GPU accelerated as well.

# References

```bibtex
@ARTICLE{Zhang2020-gi,
  title     = "{FSMI}: Fast computation of Shannon mutual information for
               information-theoretic mapping",
  author    = "Zhang, Zhengdong and Henderson, Theia and Karaman, Sertac and
               Sze, Vivienne",
  journal   = "Int. J. Rob. Res.",
  publisher = "SAGE Publications",
  volume    =  39,
  number    =  9,
  pages     = "1155--1177",
  month     =  aug,
  year      =  2020,
  url       = "http://dx.doi.org/10.1177/0278364920921941",
  doi       = "10.1177/0278364920921941",
  issn      = "0278-3649,1741-3176",
  language  = "en"
}
```

```bibtex
@ARTICLE{Asgharivaskasi2023-zb,
  title     = "Semantic {OcTree} mapping and Shannon mutual information
               computation for robot exploration",
  author    = "Asgharivaskasi, Arash and Atanasov, Nikolay",
  journal   = "IEEE Trans. Robot.",
  publisher = "Institute of Electrical and Electronics Engineers (IEEE)",
  volume    =  39,
  number    =  3,
  pages     = "1910--1928",
  month     =  jun,
  year      =  2023,
  url       = "http://dx.doi.org/10.1109/TRO.2023.3245986",
  doi       = "10.1109/tro.2023.3245986",
  issn      = "1552-3098,1941-0468",
  language  = "en"
}
```

```bibtex
@INPROCEEDINGS{Henderson2020-tq,
  title     = "An efficient and continuous approach to information-theoretic
               exploration",
  author    = "Henderson, Theia and Sze, Vivienne and Karaman, Sertac",
  booktitle = "2020 IEEE International Conference on Robotics and Automation
               (ICRA)",
  publisher = "IEEE",
  pages     = "8566--8572",
  month     =  may,
  year      =  2020,
  url       = "http://dx.doi.org/10.1109/icra40945.2020.9196592",
  doi       = "10.1109/icra40945.2020.9196592"
}
```
