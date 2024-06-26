import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

@Configuration
public class VcapServicesConfig {

    @Value("${VCAP_SERVICES:}")
    private String vcapServices;

    private JsonNode getServiceCredentials(String serviceName) throws Exception {
        if (vcapServices == null || vcapServices.isEmpty()) {
            throw new Exception("VCAP_SERVICES environment variable is not set.");
        }
        ObjectMapper mapper = new ObjectMapper();
        JsonNode root = mapper.readTree(vcapServices);
        for (JsonNode service : root.path("user-provided")) {
            if (service.path("name").asText().equals(serviceName)) {
                return service.path("credentials");
            }
        }
        throw new Exception("Service not found: " + serviceName);
    }

    public String getServiceUri(String serviceName) throws Exception {
        return getServiceCredentials(serviceName).path("uri").asText();
    }

    public String getServiceUsername(String serviceName) throws Exception {
        return getServiceCredentials(serviceName).path("username").asText();
    }

    public String getServicePassword(String serviceName) throws Exception {
        return getServiceCredentials(serviceName).path("password").asText();
    }
}