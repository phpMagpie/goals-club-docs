# Data Sources for Goals Club

## National Trust Locations

### Current Approach
We've manually curated National Trust locations for the North East region (15 properties). 

### Comprehensive UK Data Options

**Attempted Sources:**
1. ❌ **National Trust Website API** - No public API available, JavaScript-rendered SPA
2. ❌ **GitHub Repositories** - No comprehensive JSON datasets found
3. ❌ **UK Government Open Data** - No NT-specific datasets on data.gov.uk
4. ⚠️ **OpenStreetMap Overpass API** - Has NT data but queries timing out for full UK scope
5. ❌ **Wikipedia** - Lists exist but would require scraping/parsing

### Recommended Approach

**Phase 1 (Current):** Manual curation of major NT properties by region
- Focus on most visited/popular properties
- Include key property types: Houses, Gardens, Coastal, Castles, Archaeological sites
- ~15-25 properties per region
- Regions: North East, North West, Yorkshire, East Midlands, West Midlands, East England, South East, South West, Wales

**Phase 2 (Future):** Enhanced data
- Community contributions (users can suggest missing locations)
- Integrate with NT membership API if/when available
- OSM data extraction for coordinates/metadata enrichment

### Current Coverage
- ✅ **North East England** - 15 properties (Cragside, Wallington, Lindisfarne Castle, etc.)
- 🔄 **Wainwrights** - 214 peaks (complete)
- 📋 **Other regions** - TBD

### Data Quality Requirements
Each location should have:
- Name (required)
- Description (required)
- Latitude/Longitude (required for mapping)
- Address/Postcode (recommended)
- Property type/category (recommended)
- Region (required)

### Next Steps
1. Complete manual curation of 5-8 most popular NT properties per UK region
2. Deploy and test with North East goal
3. Add region-by-region as goals
4. Consider crowdsourcing additional properties from community

