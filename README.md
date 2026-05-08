#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define NUM_ZONES 100

typedef struct {
    int zone_id;
    double latitude;
    double longitude:

    double rainfall_mm;
    double humidity;
    double temperature;
    double wind_speed;
    double solar_radiation;
    double cloud_cover;

    double drain_density;
    double drain_capacity;
    int clogged_drain_reports;
    double drainage_efficiency;

    double impervious_surface_percent;

    char soil_type[20];
    double soil_absorption_score;

    double ndvi;
    double slope;
    double elevation;
    double groundwater_depth;

    int active_construction_sites;
    int excavation_count;
    int debris_complaints;
    int construction_disturbance;

    double previous_rainfall_24h;
    double previous_rainfall_7d;
    double previous_rain_saturation;

    double evaporation_potential;

    double drying_time_hr;
    char risk_level[20];

} ZoneData;

double random_double(double min, double max) {
    return min + ((double)rand() / RAND_MAX) * (max - min);
}

int random_int(int min, int max) {
    return min + rand() % (max - min + 1);
}

void assign_soil_type(ZoneData *zone) {
    int choice = random_int(1, 4);

    if (choice == 1) {
        strcpy(zone->soil_type, "sandy");
        zone->soil_absorption_score = 0.9;
    } else if (choice == 2) {
        strcpy(zone->soil_type, "loamy");
        zone->soil_absorption_score = 0.7;
    } else if (choice == 3) {
        strcpy(zone->soil_type, "mixed");
        zone->soil_absorption_score = 0.5;
    } else {
        strcpy(zone->soil_type, "clay");
        zone->soil_absorption_score = 0.2;
    }
}

void calculate_engineered_features(ZoneData *zone) {
    zone->drainage_efficiency =
        (zone->drain_density * zone->drain_capacity)
        - (0.15 * zone->clogged_drain_reports);

    zone->construction_disturbance =
        zone->active_construction_sites
        + zone->excavation_count
        + zone->debris_complaints;

    zone->previous_rain_saturation =
        zone->previous_rainfall_24h
        + (0.4 * zone->previous_rainfall_7d);

    zone->evaporation_potential =
        (0.03 * zone->temperature)
        + (0.04 * zone->wind_speed)
        + (0.002 * zone->solar_radiation)
        - (0.03 * zone->humidity)
        - (0.01 * zone->cloud_cover);
}

void calculate_drying_time(ZoneData *zone) {
    zone->drying_time_hr =
        2
        + (0.08 * zone->rainfall_mm)
        + (0.05 * zone->humidity)
        + (0.06 * zone->impervious_surface_percent)
        - (1.4 * zone->drainage_efficiency)
        - (3.0 * zone->soil_absorption_score)
        - (0.5 * zone->slope)
        - (0.18 * zone->groundwater_depth)
        + (0.18 * zone->construction_disturbance)
        + (0.025 * zone->previous_rain_saturation)
        - (1.2 * zone->evaporation_potential);

    if (zone->drying_time_hr < 1) {
        zone->drying_time_hr = 1;
    }
}

void classify_risk(ZoneData *zone) {
    if (zone->drying_time_hr <= 4) {
        strcpy(zone->risk_level, "Low");
    } else if (zone->drying_time_hr <= 12) {
        strcpy(zone->risk_level, "Medium");
    } else {
        strcpy(zone->risk_level, "High");
    }
}

void generate_zone_data(ZoneData zones[]) {
    double base_lat = 12.9716;
    double base_lon = 77.5946;

    for (int i = 0; i < NUM_ZONES; i++) {
        zones[i].zone_id = i + 1;

        zones[i].latitude = base_lat + random_double(-0.08, 0.08);
        zones[i].longitude = base_lon + random_double(-0.08, 0.08);

        zones[i].rainfall_mm = random_double(10, 90);
        zones[i].humidity = random_double(55, 98);
        zones[i].temperature = random_double(20, 34);
        zones[i].wind_speed = random_double(1, 20);
        zones[i].solar_radiation = random_double(100, 900);
        zones[i].cloud_cover = random_double(10, 95);

        zones[i].drain_density = random_double(0.5, 5.0);
        zones[i].drain_capacity = random_double(0.4, 1.0);
        zones[i].clogged_drain_reports = random_int(0, 15);

        zones[i].impervious_surface_percent = random_double(25, 95);

        assign_soil_type(&zones[i]);

        zones[i].ndvi = random_double(0.1, 0.75);
        zones[i].slope = random_double(0.2, 8.0);
        zones[i].elevation = random_double(800, 950);
        zones[i].groundwater_depth = random_double(0.5, 10.0);

        zones[i].active_construction_sites = random_int(0, 8);
        zones[i].excavation_count = random_int(0, 6);
        zones[i].debris_complaints = random_int(0, 10);

        zones[i].previous_rainfall_24h = random_double(0, 50);
        zones[i].previous_rainfall_7d = random_double(10, 250);

        calculate_engineered_features(&zones[i]);
        calculate_drying_time(&zones[i]);
        classify_risk(&zones[i]);
    }
}

void save_to_csv(ZoneData zones[]) {
    FILE *file = fopen("moisture_data.csv", "w");

    if (file == NULL) {
        printf("Error: Could not create CSV file.\n");
        return;
    }

    fprintf(file,
        "zone_id,latitude,longitude,rainfall_mm,humidity,temperature,wind_speed,"
        "solar_radiation,cloud_cover,drain_density,drain_capacity,"
        "clogged_drain_reports,drainage_efficiency,impervious_surface_percent,"
        "soil_type,soil_absorption_score,ndvi,slope,elevation,groundwater_depth,"
        "active_construction_sites,excavation_count,debris_complaints,"
        "construction_disturbance,previous_rainfall_24h,previous_rainfall_7d,"
        "previous_rain_saturation,evaporation_potential,drying_time_hr,risk_level\n"
    );

    for (int i = 0; i < NUM_ZONES; i++) {
        fprintf(file,
            "%d,%.6f,%.6f,%.2f,%.2f,%.2f,%.2f,%.2f,%.2f,"
            "%.2f,%.2f,%d,%.2f,%.2f,%s,%.2f,%.2f,%.2f,%.2f,%.2f,"
            "%d,%d,%d,%d,%.2f,%.2f,%.2f,%.2f,%.2f,%s\n",

            zones[i].zone_id,
            zones[i].latitude,
            zones[i].longitude,
            zones[i].rainfall_mm,
            zones[i].humidity,
            zones[i].temperature,
            zones[i].wind_speed,
            zones[i].solar_radiation,
            zones[i].cloud_cover,
            zones[i].drain_density,
            zones[i].drain_capacity,
            zones[i].clogged_drain_reports,
            zones[i].drainage_efficiency,
            zones[i].impervious_surface_percent,
            zones[i].soil_type,
            zones[i].soil_absorption_score,
            zones[i].ndvi,
            zones[i].slope,
            zones[i].elevation,
            zones[i].groundwater_depth,
            zones[i].active_construction_sites,
            zones[i].excavation_count,
            zones[i].debris_complaints,
            zones[i].construction_disturbance,
            zones[i].previous_rainfall_24h,
            zones[i].previous_rainfall_7d,
            zones[i].previous_rain_saturation,
            zones[i].evaporation_potential,
            zones[i].drying_time_hr,
            zones[i].risk_level
        );
    }

    fclose(file);
    printf("CSV file 'moisture_data.csv' created successfully.\n");
}

void display_zone_summary(ZoneData zone) {
    printf("\n=================================================\n");
    printf("ZONE Z%03d MOISTURE REPORT\n", zone.zone_id);
    printf("=================================================\n");

    printf("Location: %.6f, %.6f\n", zone.latitude, zone.longitude);
    printf("Rainfall: %.2f mm\n", zone.rainfall_mm);
    printf("Humidity: %.2f %%\n", zone.humidity);
    printf("Temperature: %.2f C\n", zone.temperature);
    printf("Drying Time: %.2f hours\n", zone.drying_time_hr);
    printf("Risk Level: %s\n", zone.risk_level);

    printf("\nInfrastructure and Environment:\n");
    printf("Drainage Efficiency: %.2f\n", zone.drainage_efficiency);
    printf("Clogged Drain Reports: %d\n", zone.clogged_drain_reports);
    printf("Impervious Surface: %.2f %%\n", zone.impervious_surface_percent);
    printf("Soil Type: %s\n", zone.soil_type);
    printf("Soil Absorption Score: %.2f\n", zone.soil_absorption_score);
    printf("Vegetation Index NDVI: %.2f\n", zone.ndvi);
    printf("Slope: %.2f\n", zone.slope);
    printf("Groundwater Depth: %.2f m\n", zone.groundwater_depth);
    printf("Construction Disturbance Score: %d\n", zone.construction_disturbance);
    printf("Previous Rain Saturation: %.2f\n", zone.previous_rain_saturation);
    printf("Evaporation Potential: %.2f\n", zone.evaporation_potential);
}

void explain_causes_and_recommendations(ZoneData zone) {
    printf("\nLikely Causes:\n");

    int cause_found = 0;

    if (zone.drainage_efficiency < 1.0) {
        printf("- Low drainage efficiency may be slowing water removal.\n");
        cause_found = 1;
    }

    if (zone.clogged_drain_reports > 7) {
        printf("- High clogged drain reports indicate possible blocked drainage.\n");
        cause_found = 1;
    }

    if (zone.impervious_surface_percent > 70) {
        printf("- High concrete/asphalt coverage prevents water absorption.\n");
        cause_found = 1;
    }

    if (zone.soil_absorption_score < 0.4) {
        printf("- Low soil absorption, likely due to clay-heavy soil.\n");
        cause_found = 1;
    }

    if (zone.slope < 2.0) {
        printf("- Low slope makes natural water flow difficult.\n");
        cause_found = 1;
    }

    if (zone.groundwater_depth < 2.0) {
        printf("- Shallow groundwater may keep the surface damp for longer.\n");
        cause_found = 1;
    }

    if (zone.construction_disturbance > 10) {
        printf("- Construction activity may be blocking or altering water flow.\n");
        cause_found = 1;
    }

    if (zone.previous_rain_saturation > 100) {
        printf("- Previous rainfall saturation means the area was already wet.\n");
        cause_found = 1;
    }

    if (zone.humidity > 85) {
        printf("- High humidity slows evaporation.\n");
        cause_found = 1;
    }

    if (!cause_found) {
        printf("- No major visible cause detected. Continue monitoring.\n");
    }

    printf("\nRecommended Actions:\n");

    if (zone.drainage_efficiency < 1.0 || zone.clogged_drain_reports > 7) {
        printf("- Clean and inspect local drainage channels.\n");
    }

    if (zone.impervious_surface_percent > 70) {
        printf("- Add permeable pavements, recharge pits, or rain gardens.\n");
    }

    if (zone.soil_absorption_score < 0.4) {
        printf("- Improve soil infiltration using recharge wells or soil treatment.\n");
    }

    if (zone.slope < 2.0) {
        printf("- Improve stormwater diversion in low-slope areas.\n");
    }

    if (zone.groundwater_depth < 2.0) {
        printf("- Monitor groundwater and consider subsurface drainage.\n");
    }

    if (zone.construction_disturbance > 10) {
        printf("- Remove construction debris and create temporary runoff paths.\n");
    }

    if (zone.previous_rain_saturation > 100) {
        printf("- Prepare temporary pumping after repeated rainfall.\n");
    }

    if (zone.humidity > 85) {
        printf("- Increase monitoring during humid months.\n");
    }
}

void show_high_risk_zones(ZoneData zones[]) {
    printf("\n=================================================\n");
    printf("HIGH RISK ZONES\n");
    printf("=================================================\n");

    int count = 0;

    for (int i = 0; i < NUM_ZONES; i++) {
        if (strcmp(zones[i].risk_level, "High") == 0) {
            printf("Z%03d | Drying Time: %.2f hr | Rainfall: %.2f mm | Drainage: %.2f | Impervious: %.2f%%\n",
                zones[i].zone_id,
                zones[i].drying_time_hr,
                zones[i].rainfall_mm,
                zones[i].drainage_efficiency,
                zones[i].impervious_surface_percent
            );
            count++;
        }
    }

    if (count == 0) {
        printf("No high-risk zones found.\n");
    }
}

void compare_two_zones(ZoneData zones[]) {
    int id1, id2;

    printf("\nEnter first zone number: ");
    scanf("%d", &id1);

    printf("Enter second zone number: ");
    scanf("%d", &id2);

    if (id1 < 1 || id1 > NUM_ZONES || id2 < 1 || id2 > NUM_ZONES) {
        printf("Invalid zone number.\n");
        return;
    }

    ZoneData a = zones[id1 - 1];
    ZoneData b = zones[id2 - 1];

    printf("\n=================================================\n");
    printf("ZONE COMPARISON: Z%03d vs Z%03d\n", id1, id2);
    printf("=================================================\n");

    printf("%-30s %-15s %-15s\n", "Factor", "Zone A", "Zone B");
    printf("%-30s %-15.2f %-15.2f\n", "Rainfall mm", a.rainfall_mm, b.rainfall_mm);
    printf("%-30s %-15.2f %-15.2f\n", "Drying Time hr", a.drying_time_hr, b.drying_time_hr);
    printf("%-30s %-15s %-15s\n", "Risk Level", a.risk_level, b.risk_level);
    printf("%-30s %-15.2f %-15.2f\n", "Drainage Efficiency", a.drainage_efficiency, b.drainage_efficiency);
    printf("%-30s %-15.2f %-15.2f\n", "Impervious Surface %", a.impervious_surface_percent, b.impervious_surface_percent);
    printf("%-30s %-15s %-15s\n", "Soil Type", a.soil_type, b.soil_type);
    printf("%-30s %-15.2f %-15.2f\n", "Soil Absorption", a.soil_absorption_score, b.soil_absorption_score);
    printf("%-30s %-15.2f %-15.2f\n", "Slope", a.slope, b.slope);
    printf("%-30s %-15.2f %-15.2f\n", "Groundwater Depth", a.groundwater_depth, b.groundwater_depth);
    printf("%-30s %-15d %-15d\n", "Construction Score", a.construction_disturbance, b.construction_disturbance);
}

void menu(ZoneData zones[]) {
    int choice;
    int zone_number;

    while (1) {
        printf("\n\n==============================\n");
        printf("URBAN MOISTURE INTELLIGENCE SYSTEM\n");
        printf("==============================\n");
        printf("1. Show report for a zone\n");
        printf("2. Show all high-risk zones\n");
        printf("3. Compare two zones\n");
        printf("4. Save data to CSV\n");
        printf("5. Exit\n");
        printf("Enter your choice: ");

        scanf("%d", &choice);

        switch (choice) {
            case 1:
                printf("Enter zone number between 1 and %d: ", NUM_ZONES);
                scanf("%d", &zone_number);

                if (zone_number < 1 || zone_number > NUM_ZONES) {
                    printf("Invalid zone number.\n");
                } else {
                    display_zone_summary(zones[zone_number - 1]);
                    explain_causes_and_recommendations(zones[zone_number - 1]);
                }
                break;

            case 2:
                show_high_risk_zones(zones);
                break;

            case 3:
                compare_two_zones(zones);
                break;

            case 4:
                save_to_csv(zones);
                break;

            case 5:
                printf("Exiting program.\n");
                return;

            default:
                printf("Invalid choice. Try again.\n");
        }
    }
}

int main() {
    srand(time(NULL));

    ZoneData zones[NUM_ZONES];

    generate_zone_data(zones);

    printf("Synthetic neighborhood moisture data generated successfully.\n");

    save_to_csv(zones);

    menu(zones);

    return 0;
}
