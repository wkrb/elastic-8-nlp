
# Create the eland image, which will be used to import NLP modles into elastic
```
git clone https://github.com/elastic/eland.git
cd eland/
docker build -t elastic/eland .
```

# Start up the elasticsearch cluster. This also includes kibana and our newly built eland server:
```
sudo sysctl -w vm.max_map_count=262144
docker-compose up -d
```

# To import pytorch models into our cluster:
```
docker exec -it 70aef7b4485839635bf77b81c77f7dfbf678bcae42a429a2e834b3e7c928500f bash

eland_import_hub_model --url http://es01:9200 --hub-model-id elastic/distilbert-base-cased-finetuned-conll03-english --task-type ner
eland_import_hub_model --url http://es01:9200 --hub-model-id typeform/distilbert-base-uncased-mnli --task-type zero_shot_classification
eland_import_hub_model --url http://es01:9200 --hub-model-id bhadresh-savani/bert-base-uncased-emotion --task-type text_classification

exit
```

- Then start all of the models via kibana's machine learning portal (http://localhost:5601)

# TO add a pipeline that makes use of our models:
```
PUT _ingest/pipeline/nlp-inferred-enrichments-pipeline
{
  "description": "A pipeline demonstrating multiple NLP pytorch based processors",
  "processors": [
    {
      "inference": {
        "model_id": "elastic__distilbert-base-cased-finetuned-conll03-english",
        "target_field": "ner",
        "field_map": {
          "message": "text"
        },
        "tag": "ner"
      }
    },
    {
      "set": {
        "field": "event.ingested",
        "value": "{{{_ingest.timestamp}}}"
      }
    },
    {
      "inference": {
        "model_id": "typeform__distilbert-base-uncased-mnli",
        "target_field": "mlni",
        "field_map": {
          "message": "text"
        },
        "inference_config": {
          "zero_shot_classification": {
            "labels": [
              "plan",
              "event",
              "crime",
              "weapon",
              "travel ",
              "anger",
              "violence",
              "hide"
            ]
          }
        },
        "tag": "mlni"
      }
    }
  ]
}- 
```


# Prepare our index and it's ingest pipeline

## Create the index
```
PUT tweets
{}

DELETE tweets/_mapping
```

## Add the mapping
```
PUT tweets/_mapping
{
  "properties": {
    "contributors_enabled": {
      "type": "boolean"
    },
    "created_at": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "default_profile": {
      "type": "boolean"
    },
    "default_profile_image": {
      "type": "boolean"
    },
    "description": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "display_text_range": {
      "type": "long"
    },
    "entities": {
      "properties": {
        "hashtags": {
          "properties": {
            "indices": {
              "type": "long"
            },
            "text": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "media": {
          "properties": {
            "additional_media_info": {
              "properties": {
                "description": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "embeddable": {
                  "type": "boolean"
                },
                "monetizable": {
                  "type": "boolean"
                },
                "title": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "display_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "expanded_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "id": {
              "type": "long"
            },
            "id_str": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "indices": {
              "type": "long"
            },
            "media_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "media_url_https": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "sizes": {
              "properties": {
                "large": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "medium": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "small": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "thumb": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                }
              }
            },
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "urls": {
          "properties": {
            "display_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "expanded_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "indices": {
              "type": "long"
            },
            "url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "user_mentions": {
          "properties": {
            "id": {
              "type": "long"
            },
            "id_str": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "indices": {
              "type": "long"
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "screen_name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      }
    },
    "extended_entities": {
      "properties": {
        "media": {
          "properties": {
            "additional_media_info": {
              "properties": {
                "description": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "embeddable": {
                  "type": "boolean"
                },
                "monetizable": {
                  "type": "boolean"
                },
                "title": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "display_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "expanded_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "id": {
              "type": "long"
            },
            "id_str": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "indices": {
              "type": "long"
            },
            "media_url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "media_url_https": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "sizes": {
              "properties": {
                "large": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "medium": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "small": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                },
                "thumb": {
                  "properties": {
                    "h": {
                      "type": "long"
                    },
                    "resize": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "w": {
                      "type": "long"
                    }
                  }
                }
              }
            },
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "video_info": {
              "properties": {
                "aspect_ratio": {
                  "type": "long"
                },
                "duration_millis": {
                  "type": "long"
                },
                "variants": {
                  "properties": {
                    "bitrate": {
                      "type": "long"
                    },
                    "content_type": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    },
                    "url": {
                      "type": "text",
                      "fields": {
                        "keyword": {
                          "type": "keyword",
                          "ignore_above": 256
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "extended_tweet": {
      "properties": {
        "display_text_range": {
          "type": "long"
        },
        "entities": {
          "properties": {
            "hashtags": {
              "properties": {
                "indices": {
                  "type": "long"
                },
                "text": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "user_mentions": {
              "properties": {
                "id": {
                  "type": "long"
                },
                "id_str": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "indices": {
                  "type": "long"
                },
                "name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "screen_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            }
          }
        },
        "full_text": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "favorite_count": {
      "type": "long"
    },
    "favorited": {
      "type": "boolean"
    },
    "favourites_count": {
      "type": "long"
    },
    "filter_level": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "followers_count": {
      "type": "long"
    },
    "friends_count": {
      "type": "long"
    },
    "full_text": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "geo_enabled": {
      "type": "boolean"
    },
    "hashtags": {
      "properties": {
        "indices": {
          "type": "long"
        },
        "text": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "id": {
      "type": "long"
    },
    "id_str": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "is_quote_status": {
      "type": "boolean"
    },
    "is_translator": {
      "type": "boolean"
    },
    "lang": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "listed_count": {
      "type": "long"
    },
    "location": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "media": {
      "properties": {
        "additional_media_info": {
          "properties": {
            "description": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "embeddable": {
              "type": "boolean"
            },
            "monetizable": {
              "type": "boolean"
            },
            "title": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "display_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "expanded_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "id": {
          "type": "long"
        },
        "id_str": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "indices": {
          "type": "long"
        },
        "media_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "media_url_https": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "sizes": {
          "properties": {
            "large": {
              "properties": {
                "h": {
                  "type": "long"
                },
                "resize": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "w": {
                  "type": "long"
                }
              }
            },
            "medium": {
              "properties": {
                "h": {
                  "type": "long"
                },
                "resize": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "w": {
                  "type": "long"
                }
              }
            },
            "small": {
              "properties": {
                "h": {
                  "type": "long"
                },
                "resize": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "w": {
                  "type": "long"
                }
              }
            },
            "thumb": {
              "properties": {
                "h": {
                  "type": "long"
                },
                "resize": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "w": {
                  "type": "long"
                }
              }
            }
          }
        },
        "source_status_id": {
          "type": "long"
        },
        "source_status_id_str": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "source_user_id": {
          "type": "long"
        },
        "source_user_id_str": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "type": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "video_info": {
          "properties": {
            "aspect_ratio": {
              "type": "long"
            },
            "duration_millis": {
              "type": "long"
            },
            "variants": {
              "properties": {
                "bitrate": {
                  "type": "long"
                },
                "content_type": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "url": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "possibly_sensitive": {
      "type": "boolean"
    },
    "profile_background_color": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_background_image_url": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_background_image_url_https": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_background_tile": {
      "type": "boolean"
    },
    "profile_banner_url": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_image_url": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_image_url_https": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_link_color": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_sidebar_border_color": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_sidebar_fill_color": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_text_color": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "profile_use_background_image": {
      "type": "boolean"
    },
    "protected": {
      "type": "boolean"
    },
    "quote_count": {
      "type": "long"
    },
    "reply_count": {
      "type": "long"
    },
    "retweet_count": {
      "type": "long"
    },
    "retweeted": {
      "type": "boolean"
    },
    "screen_name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "source": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "statuses_count": {
      "type": "long"
    },
    "text": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "translator_type": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "truncated": {
      "type": "boolean"
    },
    "url": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    },
    "urls": {
      "properties": {
        "display_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "expanded_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "indices": {
          "type": "long"
        },
        "url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "user": {
      "properties": {
        "contributors_enabled": {
          "type": "boolean"
        },
        "created_at": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "default_profile": {
          "type": "boolean"
        },
        "default_profile_image": {
          "type": "boolean"
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "favourites_count": {
          "type": "long"
        },
        "followers_count": {
          "type": "long"
        },
        "friends_count": {
          "type": "long"
        },
        "geo_enabled": {
          "type": "boolean"
        },
        "id": {
          "type": "long"
        },
        "id_str": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "is_translator": {
          "type": "boolean"
        },
        "lang": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "listed_count": {
          "type": "long"
        },
        "location": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_background_color": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_background_image_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_background_image_url_https": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_background_tile": {
          "type": "boolean"
        },
        "profile_banner_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_image_url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_image_url_https": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_link_color": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_sidebar_border_color": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_sidebar_fill_color": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_text_color": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "profile_use_background_image": {
          "type": "boolean"
        },
        "protected": {
          "type": "boolean"
        },
        "screen_name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "statuses_count": {
          "type": "long"
        },
        "translator_type": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "url": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "verified": {
          "type": "boolean"
        }
      }
    },
    "user_mentions": {
      "properties": {
        "id": {
          "type": "long"
        },
        "id_str": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "indices": {
          "type": "long"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "screen_name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "verified": {
      "type": "boolean"
    }
  }
}
```

## Add the pipeline
```
PUT _ingest/pipeline/twatter-ingest
{
  "description": "A pipeline demonstrating multiple NLP pytorch based processors",
  "processors" : [
      {
        "remove" : {
          "field" : [
            "contributors_enabled",
            "default_profile",
            "default_profile_image",
            "favourites_count",
            "friends_count",
            "geo_enabled",
            "id",
            "id_str",
            "is_translator",
            "listed_count",
            "location",
            "profile_background_color",
            "profile_background_image_url",
            "profile_background_image_url_https",
            "profile_background_tile",
            "profile_banner_url",
            "profile_image_url",
            "profile_image_url_https",
            "profile_link_color",
            "profile_sidebar_border_color",
            "profile_sidebar_fill_color",
            "profile_text_color",
            "profile_use_background_image",
            "protected",
            "translator_type",
            "verified"
          ]
        }
      },
      {
        "set" : {
          "field" : "text_field",
          "copy_from" : "description"
        }
      },
      {
        "inference" : {
          "model_id" : "elastic__distilbert-base-cased-finetuned-conll03-english",
          "target_field" : "moogle_ner",
          "field_map" : {
            "description" : "text"
          }
        }
      },
      {
        "inference" : {
          "model_id" : "typeform__distilbert-base-uncased-mnli",
          "target_field" : "moogle_zshots",
          "field_map" : {
            "description" : "text"
          },
          "inference_config" : {
            "zero_shot_classification" : {
              "labels" : [
                "plan",
                "event",
                "crime",
                "weapon",
                "travel ",
                "anger",
                "violence",
                "hide"
              ]
            }
          }
        }
      },
      {
        "inference" : {
          "model_id" : "bhadresh-savani__bert-base-uncased-emotion",
          "target_field" : "moogle_emotions",
          "field_map" : {
            "description" : "text"
          }
        }
      }
    ]
}
```

# Download the data sets from the below sources, and shrink it to your desired size
Tweets:
https://www.kaggle.com/xvivancos/tweets-during-r-madrid-vs-liverpool-ucl-2018 

Shakespeare plays:
https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json


# Ingest twitter data, run the following command:
```
cat Tweets2k.json | jq -c '.[] | {"index": {"_index": "tweets"}}, .' | curl -XPOST localhost:9200/_bulk?pipeline=twatter-ingest --data-binary @- -H'Content-Type: application/json'
```
# To set up the shakespeare index and mappings:
```
PUT shakespeare
{}

PUT shakespeare/_mapping
{
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
 }
}
```

# To load in the shakespheare data:
```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/_bulk?pipeline=shakespeare-ingest' --data-binary @shakespeare_6.0.json
```
# Voilà...
You you can now interrogate the enriched data in kibana. Please feel free to import my included dashboards as a starting point. They are included via the export.ndjson file.
